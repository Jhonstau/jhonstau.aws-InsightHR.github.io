---
title: "Frontend Development"
date: "2025-12-09"
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

## Frontend Development

This section covers building the React frontend application for InsightHR.

### Overview

The frontend is a Single Page Application (SPA) built with:

- **React 18** with TypeScript
- **Vite 7.2** for fast builds
- **Tailwind CSS 3.4** for styling (Frutiger Aero theme)
- **Zustand 5.0** for state management
- **React Hook Form + Zod** for form validation
- **Recharts 3.4** for data visualization
- **React Router v7** for routing

### Project Structure

```
insighthr-web/
├── src/
│   ├── components/
│   │   ├── auth/           # Login, Register, ProtectedRoute
│   │   ├── admin/          # User/Employee/Score Management
│   │   ├── dashboard/      # Charts, Tables, Filters
│   │   ├── attendance/     # Check-in/out, History
│   │   ├── chatbot/        # AI Chat Interface
│   │   ├── profile/        # User Profile
│   │   ├── common/         # Reusable UI components
│   │   └── layout/         # Header, Sidebar, Footer
│   ├── pages/              # Page components
│   ├── services/           # API service layer
│   ├── store/              # Zustand state stores
│   ├── types/              # TypeScript type definitions
│   ├── utils/              # Utility functions
│   └── styles/             # Global styles and theme
├── public/                 # Static assets
└── package.json            # Dependencies
```

### Key Components

#### Authentication Components
- **LoginForm**: Email/password and Google OAuth login
- **RegisterForm**: New user registration
- **ProtectedRoute**: Route guard for authenticated users
- **ChangePasswordModal**: Password change dialog

#### Admin Panel Components
- **UserManagement**: User CRUD with role assignment
- **EmployeeManagement**: Employee CRUD with filters
- **PerformanceScoreManagement**: Score management with bulk import
- **PasswordRequestsPanel**: Password reset request handling

#### Dashboard Components
- **PerformanceDashboard**: Main analytics dashboard
- **BarChart**: Department performance comparison
- **LineChart**: Trend analysis over time
- **PieChart**: Score distribution
- **DataTable**: Sortable, filterable data grid
- **FilterPanel**: Date range and department filters
- **ExportButton**: CSV/Excel export

#### Chatbot Components
- **MessageList**: Conversation history display
- **MessageInput**: User input with send button
- **TypingIndicator**: Loading animation
- **ChatbotInstructions**: Usage guide

#### Attendance Components
- **CheckInCheckOut**: Quick check-in/out buttons
- **AttendanceManagement**: Full attendance interface
- **AttendanceCalendarView**: Calendar visualization
- **AttendanceRecordsList**: History table

### State Management

Using Zustand for global state:

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

### API Integration

All API calls go through a centralized service layer with axios interceptors for authentication:

```typescript
const api = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL,
  headers: { 'Content-Type': 'application/json' }
});

// Add auth token to all requests
api.interceptors.request.use((config) => {
  const token = localStorage.getItem('idToken');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});
```

### Routing

React Router v7 handles navigation:

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

### Environment Configuration

Create `.env` file:

```bash
VITE_API_BASE_URL=https://your-api-id.execute-api.ap-southeast-1.amazonaws.com/dev
VITE_GOOGLE_CLIENT_ID=your-google-client-id
VITE_AWS_REGION=ap-southeast-1
VITE_COGNITO_USER_POOL_ID=your-user-pool-id
VITE_COGNITO_CLIENT_ID=your-client-id
```

### Development

```bash
# Install dependencies
npm install

# Start development server
npm run dev

# Build for production
npm run build

# Preview production build
npm run preview
```

### Styling

Tailwind CSS with custom Frutiger Aero theme provides a modern, clean interface with:
- Gradient backgrounds
- Glassmorphism effects
- Smooth animations
- Responsive design
- Accessible components

### Testing

The frontend includes:
- Component unit tests
- Integration tests for user flows
- E2E tests with Playwright
- Accessibility testing

### Next Steps

With the frontend built, proceed to [Deployment](../5.9-deployment/) to deploy the complete application to AWS.
