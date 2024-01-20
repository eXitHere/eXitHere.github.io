+++
title = 'Simple Paginate'
date = 2024-01-20T12:56:17+07:00
+++

-   ปล. เป็น Code แบบ Simple
-   ปล2. Code ตัวอย่างคือการใช้งานใน Express + Typescript + Mongoose

ใน output ที่ตอบกลับจะเป็น

```json
{
    "data": [],
    "currentPage": "หน้าปัจจุบัน",
    "nextPage": "หน้าถัดไปถ้ามี แต่ถ้าไม่มีจะเป็น Null",
    "prevPage": "หน้าก่อนหน้าถ้ามี แต่ถ้าไม่มีจะเป็น Null",
    "count": "จำนวน page ที่มี"
}
```

ซึ่งจะทำให้ Frontend สามารถนำไปใช้งานได้ง่ายยิ่งขึ้น

#### 1. สร้างไฟล์ utils/pagination.util.js

```js
interface PaginationParams {
    perPage: number;
    page: number;
    skip: number;
}

interface PaginationResponse<T> {
    data: T[];
    currentPage: number;
    nextPage?: number;
    prevPage?: number;
    count: number;
}

export function buildPaginationParam(
    page: string,
    perPage: string,
    limit: number
): PaginationParams {
    // Pagination
    const pageInt = !Number.isNaN(parseInt(page)) ? parseInt(page) : 1;
    let perPageInt = !Number.isNaN(parseInt(perPage)) ? parseInt(perPage) : 10;

    // Set a maximum limit
    perPageInt = Math.min(perPageInt, limit);

    const skip = (pageInt - 1) * perPageInt;

    return {
        perPage: perPageInt,
        page: pageInt,
        skip,
    };
}

export function buildResponsePagination<T>(
    data: T[],
    page: number,
    perPage: number,
    totalCount: number
): PaginationResponse<T> {
    const currentPage = page;
    const totalPages = Math.ceil(totalCount / perPage);

    const hasNextPage = currentPage < totalPages;
    const hasPrevPage = currentPage > 1;

    const nextPage = hasNextPage ? currentPage + 1 : undefined;
    const prevPage = hasPrevPage ? currentPage - 1 : undefined;

    return {
        data,
        currentPage,
        nextPage,
        prevPage,
        count: totalCount,
    };
}
```

#### 2. Usage

```js
const { page, perPage } = req.query

const {
    perPage: perPageInt,
    page: pageInt,
    skip,
} = buildPaginationParam(page as string, perPage as string, 10)

// Find all post
const [posts, count] = await Promise.all([
    await Post.find({})
        .skip(skip)
        .limit(perPageInt)
        .lean(),
    await Post.countDocuments(),
])

const output = buildResponsePagination(
    posts,
    pageInt,
    perPageInt,
    count
)

res.send({
    message: SuccessMessages.GetSuccess,
    data: output.data,
    currentPage: output.currentPage,
    nextPage: output.nextPage || null,
    prevPage: output.prevPage || null,
    count: count,
})
```

-   ที่ต้องกำหนด Limit ไว้ด้วย เนื่องจากป้องกันการ Dump ข้อมูลจาก database และลดการทำงานของ Server ได้
