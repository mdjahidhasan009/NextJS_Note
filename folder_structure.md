```bash
app/
├── layout.tsx
├── routing/
│   ├── (public)/
│   ├── (private)/
│   ├── admin/
│   └── api/  # auth, tRPC, hono, web hooks, og images

components/
├── ui/
└── providers/

hooks/
├── features/
│   ├── order/
│   │   ├── index.ts
│   │   ├── Order.tsx
│   │   ├── OrderProvider.tsx
│   │   ├── components/
│   │   │   ├── OrderForm.tsx
│   │   │   ├── OrderTable.tsx
│   │   │   ├── CreateOrderButton.tsx
│   │   │   └── OrderModal.tsx
│   │   └── hooks/
│   │       ├── useFetchOrders.ts
│   │       ├── useOrderMutations.ts
│   │       ├── useFetchOrderById.ts
│   │       └── useOrderStatus.ts
│   └── schema/types/

schema/
├── order.ts
└── user.ts

lib/
├── axios/
│   └── index.ts
├── datetime/
│   └── index.ts
├── postgres/
│   └── index.ts
├── redis/
│   └── index.ts
├── message-queue/
│   └── index.ts
├── your-lib/
│   └── index.ts
└── utils.ts

db/
├── schema/
│   └── index.ts
└── migrations/

api/
├── index.ts
├── routes/
├── controllers/
└── service/

```
