# PANDUAN PENULISAN KODE — Node.js + Express + MongoDB/PostgreSQL

Dokumen ini berisi standar dan konvensi penulisan kode yang wajib dipatuhi AI saat mengerjakan proyek API/backend berbasis **Node.js** dengan **Express.js**.

---

## TECH STACK

| Layer | Teknologi |
|---|---|
| Runtime | Node.js (18+) |
| Framework | Express.js |
| Language | TypeScript / JavaScript (ES6+) |
| Database | MongoDB (Mongoose) / PostgreSQL (Prisma/Knex) |
| Auth | JWT (jsonwebtoken) / Passport.js |
| Validation | Joi / Zod / express-validator |
| File Upload | Multer |
| Queue | Bull / BullMQ (Redis) |
| Deployment | VPS / Docker / Railway / Render |

---

## 1. STRUKTUR FOLDER PROJECT

```
project-root/
├── src/
│   ├── index.ts                # Entry point (app bootstrap)
│   ├── app.ts                  # Express app setup
│   │
│   ├── routes/
│   │   ├── index.ts            # Route aggregator
│   │   ├── auth.routes.ts      # Auth endpoints
│   │   ├── user.routes.ts      # User CRUD
│   │   └── item.routes.ts      # Item CRUD
│   │
│   ├── controllers/
│   │   ├── auth.controller.ts
│   │   ├── user.controller.ts
│   │   └── item.controller.ts
│   │
│   ├── services/               # Business logic
│   │   ├── auth.service.ts
│   │   └── item.service.ts
│   │
│   ├── models/                 # Database models/schemas
│   │   ├── User.ts
│   │   └── Item.ts
│   │
│   ├── middlewares/
│   │   ├── auth.middleware.ts   # JWT verification
│   │   ├── validate.middleware.ts # Request validation
│   │   └── error.middleware.ts  # Global error handler
│   │
│   ├── validations/            # Request schemas (Joi/Zod)
│   │   ├── auth.validation.ts
│   │   └── item.validation.ts
│   │
│   ├── utils/
│   │   ├── response.ts         # Standard response helper
│   │   ├── logger.ts           # Winston / Pino logger
│   │   └── constants.ts        # App constants
│   │
│   ├── config/
│   │   ├── db.ts               # Database connection
│   │   └── env.ts              # Environment validation
│   │
│   └── types/                  # TypeScript interfaces
│       └── index.ts
│
├── prisma/                     # (Jika pakai Prisma)
│   └── schema.prisma
├── tests/
│   └── item.test.ts
├── .env
├── .env.example
├── package.json
└── tsconfig.json
```

---

## 2. CONTROLLER (Slim)

Controller hanya: menerima request → validasi → panggil service → return response.

```typescript
// controllers/item.controller.ts
import { Request, Response, NextFunction } from 'express';
import { ItemService } from '@/services/item.service';
import { sendSuccess, sendError } from '@/utils/response';

export class ItemController {
  constructor(private itemService: ItemService) {}

  /**
   * Mengambil semua item dengan pagination.
   */
  getAll = async (req: Request, res: Response, next: NextFunction) => {
    try {
      const { page = 1, perPage = 10, search } = req.query;
      const result = await this.itemService.getAll({
        page: Number(page),
        perPage: Number(perPage),
        search: search as string,
      });
      return sendSuccess(res, 'Data berhasil diambil', result);
    } catch (error) {
      next(error);
    }
  };

  /**
   * Membuat item baru.
   */
  create = async (req: Request, res: Response, next: NextFunction) => {
    try {
      const item = await this.itemService.create(req.body);
      return sendSuccess(res, 'Data berhasil ditambahkan', item, 201);
    } catch (error) {
      next(error);
    }
  };
}
```

---

## 3. SERVICE LAYER

```typescript
// services/item.service.ts
import { db } from '@/config/db';

export class ItemService {
  /**
   * Mengambil semua item dengan pagination dan search.
   */
  async getAll({ page, perPage, search }: QueryParams) {
    const where = search
      ? { name: { contains: search, mode: 'insensitive' } }
      : {};

    const [data, total] = await Promise.all([
      db.item.findMany({
        where,
        skip: (page - 1) * perPage,
        take: perPage,
        orderBy: { createdAt: 'desc' },
      }),
      db.item.count({ where }),
    ]);

    return {
      data,
      meta: {
        total,
        per_page: perPage,
        current_page: page,
        last_page: Math.ceil(total / perPage),
      },
    };
  }

  /**
   * Membuat item baru dengan transaksi database.
   */
  async create(data: CreateItemDto) {
    return db.$transaction(async (tx) => {
      const item = await tx.item.create({ data });
      await tx.activityLog.create({
        data: { action: 'CREATE', entity: 'item', entityId: item.id },
      });
      return item;
    });
  }
}
```

---

## 4. STANDARD RESPONSE

```typescript
// utils/response.ts
import { Response } from 'express';

export function sendSuccess(res: Response, message: string, data?: any, statusCode = 200) {
  return res.status(statusCode).json({
    success: true,
    message,
    data: data ?? null,
  });
}

export function sendError(res: Response, message: string, statusCode = 500, error?: any) {
  return res.status(statusCode).json({
    success: false,
    message,
    error: error ?? null,
  });
}
```

---

## 5. VALIDATION MIDDLEWARE (Zod)

```typescript
// validations/item.validation.ts
import { z } from 'zod';

export const createItemSchema = z.object({
  body: z.object({
    name: z.string().min(1, 'Nama wajib diisi').max(100),
    price: z.number().int().positive('Harga harus positif'),
    isActive: z.boolean().default(true),
  }),
});

// middlewares/validate.middleware.ts
import { AnyZodObject } from 'zod';

export const validate = (schema: AnyZodObject) =>
  async (req: Request, res: Response, next: NextFunction) => {
    try {
      await schema.parseAsync({ body: req.body, query: req.query, params: req.params });
      next();
    } catch (error) {
      return sendError(res, 'Validasi gagal', 422, error.errors);
    }
  };
```

**Penggunaan di route:**
```typescript
router.post('/', validate(createItemSchema), controller.create);
```

---

## 6. AUTH MIDDLEWARE (JWT)

```typescript
// middlewares/auth.middleware.ts
import jwt from 'jsonwebtoken';

export function authMiddleware(req: Request, res: Response, next: NextFunction) {
  const token = req.headers.authorization?.replace('Bearer ', '');
  if (!token) return sendError(res, 'Token tidak ditemukan', 401);

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET!);
    req.user = decoded;
    next();
  } catch {
    return sendError(res, 'Token tidak valid', 401);
  }
}
```

---

## 7. GLOBAL ERROR HANDLER

```typescript
// middlewares/error.middleware.ts
export function errorHandler(err: Error, req: Request, res: Response, next: NextFunction) {
  console.error('[ERROR]', err.message);

  if (err.name === 'ValidationError') {
    return sendError(res, 'Validasi gagal', 422, err.message);
  }

  return sendError(res, 'Internal server error', 500,
    process.env.NODE_ENV === 'development' ? err.message : undefined
  );
}
```

---

## 8. FILE UPLOAD (Multer)

```typescript
import multer from 'multer';
import path from 'path';

const storage = multer.diskStorage({
  destination: 'uploads/',
  filename: (req, file, cb) => {
    const uniqueName = `${Date.now()}-${Math.round(Math.random() * 1e9)}${path.extname(file.originalname)}`;
    cb(null, uniqueName);
  },
});

export const upload = multer({
  storage,
  limits: { fileSize: 5 * 1024 * 1024 }, // 5 MB
  fileFilter: (req, file, cb) => {
    const allowed = ['image/jpeg', 'image/png', 'image/webp'];
    cb(null, allowed.includes(file.mimetype));
  },
});
```

---

## 9. QUEUE / BACKGROUND JOBS

```typescript
// Proses berat harus masuk queue, bukan di request cycle
import { Queue, Worker } from 'bullmq';

const emailQueue = new Queue('email', { connection: redisConfig });

// Dispatch job
await emailQueue.add('send-invoice', { orderId: order.id, email: user.email });

// Worker (file terpisah)
const worker = new Worker('email', async (job) => {
  await sendEmail(job.data.email, generateInvoice(job.data.orderId));
}, { connection: redisConfig });
```

---

## 10. ENVIRONMENT

```env
# .env.example
NODE_ENV=development
PORT=3000
DATABASE_URL="postgresql://user:pass@localhost:5432/dbname"
JWT_SECRET="random-secret-min-32-chars"
REDIS_URL="redis://localhost:6379"
```

- Validasi env saat startup:
```typescript
// config/env.ts
const requiredEnvVars = ['DATABASE_URL', 'JWT_SECRET'];
requiredEnvVars.forEach((key) => {
  if (!process.env[key]) throw new Error(`Missing env: ${key}`);
});
```
