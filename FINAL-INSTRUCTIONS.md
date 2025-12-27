# FINAL Implementation Instructions - Academic Portal

## Complete Step-by-Step Guide

### Step 1: Clone & Setup (3 minutes)

```bash
git clone https://github.com/Academic-Portal-source/supabase-academic-portal.git
cd supabase-academic-portal
npm install
```

### Step 2: Create Local Project with React

```bash
npm create vite@latest academic-portal -- --template react
cd academic-portal  
npm install @supabase/supabase-js react-router-dom lucide-react
```

### Step 3: Environment Variables

Create `.env.local`:
```
VITE_SUPABASE_URL=YOUR_SUPABASE_URL
VITE_SUPABASE_ANON_KEY=YOUR_ANON_KEY
```

### Step 4: Setup Supabase Database

1. Create project at supabase.com
2. Go to SQL Editor
3. Copy-paste **ENTIRE** SQL from SETUP-GUIDE.md
4. Execute all SQL

### Step 5: Directory Structure

Create this structure in `src/`:

```
src/
├── lib/
│   └── supabase.js
├── pages/
│   ├── Login.jsx
│   ├── AdminDashboard.jsx
│   ├── StudentDashboard.jsx
│   ├── ManageStudents.jsx
│   ├── ManageCourses.jsx
│   ├── ManageAssessments.jsx
│   └── GradeStudents.jsx
├── components/
│   ├── Navbar.jsx
│   └── PrivateRoute.jsx
├── styles/
│   ├── Auth.css
│   ├── Dashboard.css
│   ├── Management.css
│   └── Navbar.css
├── App.jsx
└── App.css
```

### Step 6: Copy Code Files

Refer to FULL-CODE.md for all file content.

Key files:
- **src/lib/supabase.js** - Supabase client init
- **src/App.jsx** - Main router setup
- **src/pages/Login.jsx** - Auth page with admin/student signup
- **src/pages/AdminDashboard.jsx** - Admin home with stats
- **src/pages/StudentDashboard.jsx** - Student assessments
- **src/pages/ManageStudents.jsx** - Add/remove students
- **src/pages/ManageCourses.jsx** - Course CRUD
- **src/pages/ManageAssessments.jsx** - Assessment CRUD
- **src/pages/GradeStudents.jsx** - Mark assessments
- **src/components/Navbar.jsx** - Navigation
- **src/components/PrivateRoute.jsx** - Route guards

### Step 7: Run Application

```bash
npm run dev
```

Open: http://localhost:5173

### Step 8: Create Admin Account

1. Sign up with: **admin@example.com** / **admin123**
2. Go to Supabase Dashboard → profiles table
3. Find admin@example.com row → edit role → set to 'admin'
4. Login again as admin

## Complete Page Code Examples

### pages/Login.jsx (Shortened)

```jsx
import { useState } from 'react';
import { useNavigate } from 'react-router-dom';
import { supabase } from '../lib/supabase';
import '../styles/Auth.css';

export default function Login() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [isSignUp, setIsSignUp] = useState(false);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState('');
  const navigate = useNavigate();

  const handleAuth = async (e) => {
    e.preventDefault();
    setLoading(true);
    setError('');

    try {
      if (isSignUp) {
        const { error: err } = await supabase.auth.signUp({ email, password });
        if (err) throw err;
        setError('✅ Success! Please log in now.');
        setEmail('');
        setPassword('');
        setIsSignUp(false);
      } else {
        const { data, error: err } = await supabase.auth.signInWithPassword({ email, password });
        if (err) throw err;
        const { data: profile } = await supabase.from('profiles').select('role').eq('id', data.user.id).single();
        navigate(profile.role === 'admin' ? '/admin' : '/student');
      }
    } catch (err) {
      setError('❌ ' + err.message);
    }
    setLoading(false);
  };

  return (
    <div className="auth-container">
      <div className="auth-box">
        <h1>📚 Academic Portal</h1>
        <form onSubmit={handleAuth}>
          <div className="form-group">
            <label>Email</label>
            <input type="email" value={email} onChange={(e) => setEmail(e.target.value)} required />
          </div>
          <div className="form-group">
            <label>Password</label>
            <input type="password" value={password} onChange={(e) => setPassword(e.target.value)} required />
          </div>
          <button disabled={loading}>{loading ? 'Loading...' : isSignUp ? 'Sign Up' : 'Login'}</button>
        </form>
        {error && <p className={error.includes('✅') ? 'success' : 'error'}>{error}</p>}
      </div>
    </div>
  );
}
```

### components/PrivateRoute.jsx

```jsx
import { Navigate, Outlet } from 'react-router-dom';

export default function PrivateRoute({ user, role }) {
  if (!user) return <Navigate to="/login" />;
  return <Outlet />;
}
```

### components/Navbar.jsx

```jsx
import { useNavigate } from 'react-router-dom';
import { supabase } from '../lib/supabase';
import '../styles/Navbar.css';

export default function Navbar({ role }) {
  const navigate = useNavigate();

  return (
    <nav className="navbar">
      <div className="nav-brand">📚 Academic Portal</div>
      <div className="nav-menu">
        {role === 'admin' && (
          <>
            <button onClick={() => navigate('/admin')}>Dashboard</button>
            <button onClick={() => navigate('/admin/students')}>Students</button>
            <button onClick={() => navigate('/admin/courses')}>Courses</button>
            <button onClick={() => navigate('/admin/assessments')}>Assessments</button>
            <button onClick={() => navigate('/admin/grades')}>Grades</button>
          </>
        )}
        {role === 'student' && <button onClick={() => navigate('/student')}>My Work</button>}
        <button onClick={() => supabase.auth.signOut().then(() => navigate('/login'))}>Logout</button>
      </div>
    </nav>
  );
}
```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| "Invalid login" | Check .env.local variables. Verify user exists in auth.users |
| "RLS policy error" | Re-run ALL SQL from SETUP-GUIDE.md. Check is_admin() function exists |
| "Students not showing" | Verify profiles table has trigger. Create student via signup |
| "Admin can't see dashboard" | Update profile role to 'admin' in Supabase |

## Features Included

✅ Complete authentication (signup/login)
✅ Role-based access control (admin/student)  
✅ Student management (CRUD)
✅ Course management (CRUD)
✅ Assessment creation (quiz/assignment/exam)
✅ Grading system with feedback
✅ Student dashboard with assessments
✅ Responsive design
✅ Security with Row Level Security
✅ Modern UI with gradient backgrounds

## Next Steps

1. ✅ Clone repo
2. ✅ Copy code from FULL-CODE.md
3. ✅ Setup Supabase SQL
4. ✅ Create .env.local
5. ✅ Run `npm run dev`
6. ✅ Sign up as admin@example.com
7. ✅ Set role to admin in Supabase
8. ✅ Login and start using!

**Everything is ready. Just copy the code and run!** 🚀
