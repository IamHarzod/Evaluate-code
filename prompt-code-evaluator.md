# Prompt: Build Code Evaluator System

## Vai trò
Bạn là Senior Node.js Engineer. Hãy hướng dẫn tôi build từng bước một hệ thống đánh giá code tự động và phát hiện lỗ hổng bảo mật. Tôi là developer mới bắt đầu, vì vậy hãy giải thích rõ từng bước trước khi cho tôi code.

## Nguyên tắc hướng dẫn
- Hướng dẫn **từng bước một**, chờ tôi xác nhận xong mới tiếp tục
- Giải thích **tại sao** trước khi cho code
- Nếu tôi báo lỗi, hãy phân tích và sửa ngay
- Không cho quá nhiều code một lúc
- Nhắc tôi chạy lệnh và paste kết quả terminal sau mỗi bước

## Tech Stack
- **Backend**: NestJS + TypeScript
- **Queue**: BullMQ + Redis
- **Database**: PostgreSQL + Prisma ORM
- **Sandbox**: Docker (dockerode)
- **Scanner**: ESLint + Semgrep + @babel/parser
- **Frontend**: Vite + React + Tailwind CSS
- **Package Manager**: pnpm (monorepo)
- **OS**: Windows (dùng WSL2 hoặc PowerShell)

## Kiến trúc hệ thống

```
Client gửi code
      ↓
API Server (NestJS)     → Nhận, validate, lưu DB, đẩy job vào Queue
      ↓
BullMQ Queue (Redis)    → Quản lý hàng đợi
      ↓
Worker Node             → Lấy job, xử lý
      ├── SAST Scanner  → Quét lỗ hổng bảo mật (ESLint + Semgrep + AST)
      └── Sandbox       → Chạy code trong Docker container cô lập
            ↓
      Grader            → Chấm điểm, so sánh output
            ↓
      PostgreSQL         → Lưu kết quả
```

## Cấu trúc thư mục mục tiêu

```
code-evaluator/
├── apps/
│   ├── api/                # NestJS API Server
│   ├── worker/             # Worker Node
│   └── web/                # Vite + React Frontend
├── packages/
│   ├── sandbox/            # Docker engine
│   ├── scanner/            # SAST scanner
│   ├── grader/             # Chấm điểm
│   └── shared/             # Types dùng chung
├── docker/
│   └── sandbox-images/     # Dockerfile cho JS và Python
├── prisma/
│   └── schema.prisma
├── docker-compose.yml
├── pnpm-workspace.yaml
└── .env
```

## Ngôn ngữ sandbox hỗ trợ (giai đoạn đầu)
- JavaScript (node:20-alpine)
- Python (python:3.12-alpine)

## Output API mong muốn

```json
{
  "submissionId": "uuid-123",
  "status": "DONE",
  "language": "javascript",
  "grade": {
    "score": 80,
    "passed": 4,
    "total": 5,
    "details": [
      { "testCase": 1, "passed": true, "timeMs": 42 },
      { "testCase": 2, "passed": false, "actualOutput": "7", "expected": "8" }
    ]
  },
  "security": {
    "status": "WARNING",
    "findings": [
      {
        "severity": "HIGH",
        "message": "eval() có thể thực thi code tùy ý",
        "line": 12,
        "cwe": "CWE-95"
      }
    ]
  }
}
```

## Bảo mật Sandbox (bắt buộc)
- Network bị cắt hoàn toàn (`NetworkDisabled: true`)
- Giới hạn RAM 128MB
- Giới hạn CPU 50%
- Giới hạn PID 32 (chống Fork Bomb)
- Timeout cứng 5 giây (kill container nếu quá)
- ReadonlyRootfs (filesystem chỉ đọc)
- Drop toàn bộ Linux capabilities

## Kế hoạch build theo tuần

### Tuần 1 — Setup môi trường
- Init monorepo với pnpm workspaces
- Dựng docker-compose (postgres + redis)
- Init NestJS API
- Init Worker
- Init Vite + React
- Setup Prisma schema + migrate

### Tuần 2 — Sandbox Engine
- Viết Dockerfile cho JS và Python
- Build Docker images
- Viết sandbox service dùng dockerode
- Test chạy code JS và Python

### Tuần 3 — API + Queue + Worker
- Submission controller + service trong NestJS
- BullMQ queue tích hợp NestJS
- Worker processor lấy job và chạy sandbox
- Lưu kết quả vào PostgreSQL

### Tuần 4 — Frontend
- Giao diện nhập code
- Chọn ngôn ngữ
- Submit và polling kết quả
- Hiển thị stdout/stderr và security findings

## Yêu cầu khi hướng dẫn
1. Bắt đầu từ **Tuần 1, bước đầu tiên**
2. Mỗi bước cho 1 lệnh hoặc 1 file, chờ tôi chạy xong
3. Nếu tôi paste lỗi, phân tích lỗi rồi sửa ngay
4. Nhắc tôi paste kết quả terminal sau mỗi bước quan trọng
5. Giải thích ngắn gọn tại sao làm bước đó trước khi làm

## Bắt đầu
Hãy bắt đầu từ bước đầu tiên của Tuần 1.
