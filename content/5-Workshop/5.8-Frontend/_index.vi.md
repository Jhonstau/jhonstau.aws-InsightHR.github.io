---
title: "Phát triển Frontend"
date: "2025-12-09"
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

## Phát triển Frontend

Phần này bao gồm xây dựng ứng dụng frontend React cho InsightHR.

### Tổng quan

Frontend là một Single Page Application (SPA) được xây dựng với:

- **React 18** với TypeScript
- **Vite 7.2** cho build nhanh
- **Tailwind CSS 3.4** cho styling (theme Frutiger Aero)
- **Zustand 5.0** cho quản lý state
- **React Hook Form + Zod** cho xác thực form
- **Recharts 3.4** cho trực quan hóa dữ liệu
- **React Router v7** cho routing

### Cấu trúc Dự án

```
insighthr-web/
├── src/
│   ├── components/
│   │   ├── auth/           # Login, Register, ProtectedRoute
│   │   ├── admin/          # Quản lý User/Employee/Score
│   │   ├── dashboard/      # Charts, Tables, Filters
│   │   ├── attendance/     # Check-in/out, History
│   │   ├── chatbot/        # Giao diện AI Chat
│   │   ├── profile/        # Hồ sơ Người dùng
│   │   ├── common/         # Components UI tái sử dụng
│   │   └── layout/         # Header, Sidebar, Footer
│   ├── pages/              # Components trang
│   ├── services/           # Tầng dịch vụ API
│   ├── store/              # Zustand state stores
│   ├── types/              # Định nghĩa kiểu TypeScript
│   ├── utils/              # Hàm tiện ích
│   └── styles/             # Styles toàn cục và theme
├── public/                 # Tài nguyên tĩnh
└── package.json            # Dependencies
```

### Components Chính

#### Authentication Components
- **LoginForm**: Đăng nhập email/password và Google OAuth
- **RegisterForm**: Đăng ký người dùng mới
- **ProtectedRoute**: Route guard cho người dùng đã xác thực
- **ChangePasswordModal**: Dialog đổi mật khẩu

#### Admin Panel Components
- **UserManagement**: CRUD người dùng với gán vai trò
- **EmployeeManagement**: CRUD nhân viên với filters
- **PerformanceScoreManagement**: Quản lý điểm với import hàng loạt
- **PasswordRequestsPanel**: Xử lý yêu cầu đặt lại mật khẩu

#### Dashboard Components
- **PerformanceDashboard**: Dashboard phân tích chính
- **BarChart**: So sánh hiệu suất phòng ban
- **LineChart**: Phân tích xu hướng theo thời gian
- **PieChart**: Phân phối điểm
- **DataTable**: Lưới dữ liệu có thể sắp xếp, lọc
- **FilterPanel**: Filters khoảng thời gian và phòng ban
- **ExportButton**: Xuất CSV/Excel

#### Chatbot Components
- **MessageList**: Hiển thị lịch sử hội thoại
- **MessageInput**: Đầu vào người dùng với nút gửi
- **TypingIndicator**: Animation loading
- **ChatbotInstructions**: Hướng dẫn sử dụng

#### Attendance Components
- **CheckInCheckOut**: Nút check-in/out nhanh
- **AttendanceManagement**: Giao diện chấm công đầy đủ
- **AttendanceCalendarView**: Trực quan hóa lịch
- **AttendanceRecordsList**: Bảng lịch sử

### Quản lý State

Sử dụng Zustand cho state toàn cục:

```typescript
// Auth Store
interface AuthState {
  user: User | null;
  tokens: Tokens | null;
  isAuthenticated: boolean;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
}

// Employee Store
interface EmployeeState {
  employees: Employee[];
  loading: boolean;
  error: string | null;
  fetchEmployees: (filters?: Filters) => Promise<void>;
  createEmployee: (data: EmployeeInput) => Promise<void>;
}
```

### Tích hợp API

Tất cả các lời gọi API đi qua một tầng dịch vụ tập trung với axios interceptors cho xác thực:

```typescript
const api = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL,
  headers: { 'Content-Type': 'application/json' }
});

// Thêm auth token vào tất cả requests
api.interceptors.request.use((config) => {
  const token = localStorage.getItem('idToken');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});
```

### Routing

React Router v7 xử lý điều hướng:

```typescript
const router = createBrowserRouter([
  { path: '/login', element: <LoginPage /> },
  { path: '/register', element: <RegisterForm /> },
  {
    path: '/',
    element: <ProtectedRoute><Layout /></ProtectedRoute>,
    children: [
      { path: '/', element: <DashboardPage /> },
      { path: '/admin', element: <AdminPage /> },
      { path: '/employees', element: <EmployeesPage /> },
      { path: '/performance-scores', element: <PerformanceScoresPage /> },
      { path: '/attendance', element: <AttendancePage /> },
      { path: '/chatbot', element: <ChatbotPage /> },
      { path: '/profile', element: <ProfilePage /> }
    ]
  }
]);
```

### Cấu hình Môi trường

Tạo file `.env`:

```bash
VITE_API_BASE_URL=https://your-api-id.execute-api.ap-southeast-1.amazonaws.com/dev
VITE_GOOGLE_CLIENT_ID=your-google-client-id
VITE_AWS_REGION=ap-southeast-1
VITE_COGNITO_USER_POOL_ID=your-user-pool-id
VITE_COGNITO_CLIENT_ID=your-client-id
```

### Development

```bash
# Cài đặt dependencies
npm install

# Khởi động development server
npm run dev

# Build cho production
npm run build

# Xem trước production build
npm run preview
```

### Styling

Tailwind CSS với theme Frutiger Aero tùy chỉnh cung cấp giao diện hiện đại, sạch sẽ với:
- Nền gradient
- Hiệu ứng glassmorphism
- Animations mượt mà
- Thiết kế responsive
- Components có khả năng truy cập

### Kiểm tra

Frontend bao gồm:
- Unit tests cho components
- Integration tests cho luồng người dùng
- E2E tests với Playwright
- Kiểm tra khả năng truy cập

### Bước Tiếp theo

Với frontend đã được xây dựng, tiến hành [Triển khai](../5.9-deployment/) để triển khai ứng dụng hoàn chỉnh lên AWS.
