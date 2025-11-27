Ошибка "Учетные данные не были предоставлены" означает, что фронтенд не отправляет аутентификационные данные (сессию или токен) при запросе к API. Давайте это исправим.

Решение 1: Настройка CORS и сессионной аутентификации

1. Обновите настройки Django karate_core/settings.py:

```python
# Добавьте в конец settings.py

# CORS настройки
CORS_ALLOWED_ORIGINS = [
    "http://localhost:3000",
    "http://127.0.0.1:3000",
]

CORS_ALLOW_CREDENTIALS = True

# Для разработки можно также разрешить все (не для продакшена)
# CORS_ALLOW_ALL_ORIGINS = True

# Сессионная аутентификация для API
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.SessionAuthentication',
        'rest_framework.authentication.BasicAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticatedOrReadOnly',
    ],
}

# Настройки сессии
SESSION_COOKIE_SAMESITE = 'Lax'
SESSION_COOKIE_HTTPONLY = True
SESSION_COOKIE_AGE = 1209600  # 2 недели
```

2. Установите django-cors-headers если еще не установлен:

```bash
pip install django-cors-headers
```

Добавьте в INSTALLED_APPS:

```python
INSTALLED_APPS = [
    # ...
    'corsheaders',
]
```

И в MIDDLEWARE:

```python
MIDDLEWARE = [
    'corsheaders.middleware.CorsMiddleware',  # Должен быть первым
    # ...
]
```

3. Обновите API сервис во фронтенде src/services/api.ts:

```typescript
import axios from 'axios';
import { Tournament, Participant, Judge, Bracket, Match, Registration } from '../types';

const API_BASE_URL = 'http://localhost:8000/api';

const api = axios.create({
  baseURL: API_BASE_URL,
  headers: {
    'Content-Type': 'application/json',
  },
  withCredentials: true,  // Важно: отправляем куки с каждым запросом
});

// Интерцептор для логирования запросов (для отладки)
api.interceptors.request.use(
  (config) => {
    console.log(`Making ${config.method?.toUpperCase()} request to: ${config.url}`);
    return config;
  },
  (error) => {
    return Promise.reject(error);
  }
);

// Интерцептор для обработки ошибок
api.interceptors.response.use(
  (response) => response,
  (error) => {
    console.error('API Error:', error.response?.data || error.message);
    
    // Если ошибка аутентификации, перенаправляем на страницу логина
    if (error.response?.status === 401) {
      console.warn('Authentication required. Redirect to login.');
      // Здесь можно добавить перенаправление на страницу логина
      window.location.href = '/admin/login/?next=/';
    }
    
    return Promise.reject(error);
  }
);

// Экспортируем API функции
export const tournamentAPI = {
  getAll: () => api.get<Tournament[]>('/tournaments/tournaments/'),
  getById: (id: number) => api.get<Tournament>(`/tournaments/tournaments/${id}/`),
  create: (data: Partial<Tournament>) => api.post<Tournament>('/tournaments/tournaments/', data),
  update: (id: number, data: Partial<Tournament>) => api.put<Tournament>(`/tournaments/tournaments/${id}/`, data),
  delete: (id: number) => api.delete(`/tournaments/tournaments/${id}/`),
};

// ... остальные API функции остаются без изменений
```

Решение 2: Временное отключение аутентификации (для разработки)

Если вы хотите быстро протестировать функционал, можно временно отключить аутентификацию в Django:

В karate_core/settings.py:

```python
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.SessionAuthentication',
        'rest_framework.authentication.BasicAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.AllowAny',  # Разрешаем доступ без аутентификации
    ],
}
```

Решение 3: Добавление системы аутентификации во фронтенде

Создадим компонент для логина:

1. Создайте файл src/components/common/Login.tsx:

```typescript
import React, { useState } from 'react';
import {
  Dialog,
  DialogTitle,
  DialogContent,
  TextField,
  DialogActions,
  Button,
  Box,
  Typography,
} from '@mui/material';

interface LoginProps {
  open: boolean;
  onClose: () => void;
  onLogin: (username: string, password: string) => void;
}

const Login: React.FC<LoginProps> = ({ open, onClose, onLogin }) => {
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');

  const handleSubmit = () => {
    onLogin(username, password);
    setUsername('');
    setPassword('');
  };

  return (
    <Dialog open={open} onClose={onClose}>
      <DialogTitle>Вход в систему</DialogTitle>
      <DialogContent>
        <Box sx={{ pt: 1 }}>
          <TextField
            autoFocus
            margin="dense"
            label="Имя пользователя"
            type="text"
            fullWidth
            variant="outlined"
            value={username}
            onChange={(e) => setUsername(e.target.value)}
          />
          <TextField
            margin="dense"
            label="Пароль"
            type="password"
            fullWidth
            variant="outlined"
            value={password}
            onChange={(e) => setPassword(e.target.value)}
          />
        </Box>
        <Typography variant="body2" color="text.secondary" sx={{ mt: 2 }}>
          Используйте учетные данные из Django админки
        </Typography>
      </DialogContent>
      <DialogActions>
        <Button onClick={onClose}>Отмена</Button>
        <Button onClick={handleSubmit} variant="contained">
          Войти
        </Button>
      </DialogActions>
    </Dialog>
  );
};

export default Login;
```

2. Обновите App.tsx для обработки аутентификации:

```typescript
import React, { useState, useEffect } from 'react';
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';
import { ThemeProvider, createTheme } from '@mui/material/styles';
import CssBaseline from '@mui/material/CssBaseline';

import Layout from './components/common/Layout';
import Login from './components/common/Login';
import Dashboard from './pages/Dashboard';
import TournamentsPage from './pages/TournamentsPage';
import ParticipantsPage from './pages/ParticipantsPage';
import JudgesPage from './pages/JudgesPage';
import BracketsPage from './pages/BracketsPage';
import ScoringPage from './pages/ScoringPage';
import api from './services/api';

const theme = createTheme({
  palette: {
    primary: {
      main: '#1976d2',
    },
    secondary: {
      main: '#dc004e',
    },
  },
});

function App() {
  const [isAuthenticated, setIsAuthenticated] = useState<boolean | null>(null);
  const [loginOpen, setLoginOpen] = useState(false);

  useEffect(() => {
    // Проверяем аутентификацию при загрузке приложения
    checkAuthentication();
  }, []);

  const checkAuthentication = async () => {
    try {
      // Можно сделать запрос к защищенному эндпоинту для проверки аутентификации
      await api.get('/tournaments/tournaments/');
      setIsAuthenticated(true);
    } catch (error) {
      setIsAuthenticated(false);
      setLoginOpen(true);
    }
  };

  const handleLogin = async (username: string, password: string) => {
    try {
      // Логин через Django сессии
      const formData = new FormData();
      formData.append('username', username);
      formData.append('password', password);

      await api.post('/api-auth/login/', formData, {
        headers: {
          'Content-Type': 'multipart/form-data',
        },
      });

      setIsAuthenticated(true);
      setLoginOpen(false);
    } catch (error) {
      alert('Ошибка входа. Проверьте учетные данные.');
      console.error('Login error:', error);
    }
  };

  const handleLogout = async () => {
    try {
      await api.post('/api-auth/logout/');
      setIsAuthenticated(false);
      setLoginOpen(true);
    } catch (error) {
      console.error('Logout error:', error);
    }
  };

  if (isAuthenticated === null) {
    // Пока проверяем аутентификацию, показываем заглушку
    return <div>Загрузка...</div>;
  }

  return (
    <ThemeProvider theme={theme}>
      <CssBaseline />
      <Router>
        {isAuthenticated ? (
          <Layout onLogout={handleLogout}>
            <Routes>
              <Route path="/" element={<Dashboard />} />
              <Route path="/tournaments" element={<TournamentsPage />} />
              <Route path="/tournaments/:id" element={<TournamentsPage />} />
              <Route path="/participants" element={<ParticipantsPage />} />
              <Route path="/judges" element={<JudgesPage />} />
              <Route path="/brackets" element={<BracketsPage />} />
              <Route path="/scoring" element={<ScoringPage />} />
            </Routes>
          </Layout>
        ) : (
          <Login
            open={loginOpen}
            onClose={() => setLoginOpen(false)}
            onLogin={handleLogin}
          />
        )}
      </Router>
    </ThemeProvider>
  );
}

export default App;
```

3. Обновите Layout.tsx для поддержки выхода:

```typescript
// Добавьте в импорты
import { Button } from '@mui/material';
import { Logout } from '@mui/icons-material';

// Добавьте в интерфейс LayoutProps
interface LayoutProps {
  children: ReactNode;
  onLogout?: () => void;
}

// В компоненте Layout добавьте кнопку выхода в AppBar
<AppBar position="fixed" sx={{ zIndex: (theme) => theme.zIndex.drawer + 1 }}>
  <Toolbar>
    <MartialArtsIcon sx={{ mr: 2 }} />
    <Typography variant="h6" noWrap component="div" sx={{ flexGrow: 1 }}>
      Система проведения соревнований по каратэ
    </Typography>
    {onLogout && (
      <Button color="inherit" onClick={onLogout} startIcon={<Logout />}>
        Выйти
      </Button>
    )}
  </Toolbar>
</AppBar>
```

Быстрое решение для тестирования:

Самый простой способ протестировать - открыть Django админку в том же браузере:

1. Откройте http://localhost:8000/admin
2. Войдите под учетной записью администратора
3. Теперь откройте http://localhost:3000
4. Сессия будет общей, и фронтенд сможет делать запросы к API

Рекомендую начать с Решение 1 (настройка CORS и сессий), так как это наиболее правильный подход. Если будут проблемы, временно используйте Решение 2 для разработки.
