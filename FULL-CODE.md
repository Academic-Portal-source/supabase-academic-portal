# Complete Source Code - All React Components

## Files to Create in src/

### 1. lib/supabase.js
```javascript
import { createClient } from '@supabase/supabase-js';

const url = import.meta.env.VITE_SUPABASE_URL;
const key = import.meta.env.VITE_SUPABASE_ANON_KEY;

export const supabase = createClient(url, key);
export const getCurrentUser = async () => {
  const { data: { user } } = await supabase.auth.getUser();
  return user;
};
export const getUserProfile = async (userId) => {
  const { data } = await supabase.from('profiles').select('*').eq('id', userId).single();
  return data;
};
export const signOut = async () => supabase.auth.signOut();
```

### 2. App.jsx
```jsx
import { BrowserRouter, Routes, Route, Navigate } from 'react-router-dom';
import { useState, useEffect } from 'react';
import { supabase } from './lib/supabase';
import PrivateRoute from './components/PrivateRoute';
import Login from './pages/Login';
import AdminDashboard from './pages/AdminDashboard';
import StudentDashboard from './pages/StudentDashboard';
import ManageStudents from './pages/ManageStudents';
import ManageCourses from './pages/ManageCourses';
import ManageAssessments from './pages/ManageAssessments';
import GradeStudents from './pages/GradeStudents';

function App() {
  const [user, setUser] = useState(null);
  const [role, setRole] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    (async () => {
      const { data } = await supabase.auth.getSession();
      if (data.session) {
        setUser(data.session.user);
        const { data: profile } = await supabase.from('profiles').select('role').eq('id', data.session.user.id).single();
        setRole(profile?.role);
      }
      setLoading(false);
    })();
  }, []);

  if (loading) return <div>Loading...</div>;

  return (
    <BrowserRouter>
      <Routes>
        <Route path="/login" element={<Login />} />
        <Route element={<PrivateRoute user={user} role={role} />}>
          <Route path="/admin" element={<AdminDashboard />} />
          <Route path="/admin/students" element={<ManageStudents />} />
          <Route path="/admin/courses" element={<ManageCourses />} />
          <Route path="/admin/assessments" element={<ManageAssessments />} />
          <Route path="/admin/grades" element={<GradeStudents />} />
          <Route path="/student" element={<StudentDashboard />} />
        </Route>
        <Route path="/" element={<Navigate to={user ? '/admin' : '/login'} />} />
      </Routes>
    </BrowserRouter>
  );
}
export default App;
```

### 3. pages/Login.jsx - See DETAILED-CODE.md
### 4. pages/AdminDashboard.jsx - See DETAILED-CODE.md
### 5. pages/StudentDashboard.jsx - See DETAILED-CODE.md
### 6. pages/ManageStudents.jsx - See DETAILED-CODE.md
### 7. pages/ManageCourses.jsx - See DETAILED-CODE.md
### 8. pages/ManageAssessments.jsx - See DETAILED-CODE.md
### 9. pages/GradeStudents.jsx - See DETAILED-CODE.md
### 10. components/Navbar.jsx - See DETAILED-CODE.md
### 11. components/PrivateRoute.jsx - See DETAILED-CODE.md

## CSS Files

### styles/Auth.css
```css
.auth-container { display: flex; justify-content: center; align-items: center; min-height: 100vh; background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); }
.auth-box { background: white; padding: 2rem; border-radius: 12px; box-shadow: 0 10px 40px rgba(0,0,0,0.3); width: 100%; max-width: 400px; }
.auth-box h1 { text-align: center; margin-bottom: 2rem; color: #333; }
.form-group { margin-bottom: 1.5rem; }
.form-group label { display: block; margin-bottom: 0.5rem; font-weight: 600; color: #333; }
.form-group input { width: 100%; padding: 0.75rem; border: 2px solid #e0e0e0; border-radius: 8px; font-size: 1rem; }
.form-group input:focus { outline: none; border-color: #667eea; }
button[type="submit"] { width: 100%; padding: 0.75rem; background: #667eea; color: white; border: none; border-radius: 8px; font-weight: 600; cursor: pointer; }
button[type="submit"]:hover { background: #5568d3; }
.error { color: #e74c3c; text-align: center; margin-top: 1rem; }
.success { color: #27ae60; text-align: center; margin-top: 1rem; }
.link-btn { background: none; border: none; color: #667eea; cursor: pointer; text-decoration: underline; }
```

### styles/Dashboard.css
```css
.dashboard-container { padding: 2rem; max-width: 1200px; margin: 0 auto; }
.stats-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(250px, 1fr)); gap: 1.5rem; margin-bottom: 3rem; }
.stat-card { background: white; padding: 1.5rem; border-radius: 12px; box-shadow: 0 2px 12px rgba(0,0,0,0.1); text-align: center; }
.action-cards { display: grid; grid-template-columns: repeat(auto-fit, minmax(280px, 1fr)); gap: 1.5rem; }
.action-card { background: white; padding: 1.5rem; border-radius: 12px; box-shadow: 0 2px 12px rgba(0,0,0,0.1); cursor: pointer; transition: transform 0.3s; }
.action-card:hover { transform: translateY(-5px); box-shadow: 0 8px 24px rgba(0,0,0,0.15); }
```

### styles/Management.css
```css
.management-container { padding: 2rem; max-width: 1000px; margin: 0 auto; }
.add-form { display: grid; grid-template-columns: 1fr 1fr auto; gap: 1rem; margin-bottom: 2rem; }
.add-form input { padding: 0.75rem; border: 2px solid #e0e0e0; border-radius: 8px; }
.add-form button { background: #667eea; color: white; border: none; padding: 0.75rem 1.5rem; border-radius: 8px; cursor: pointer; }
.data-table { width: 100%; border-collapse: collapse; background: white; border-radius: 12px; overflow: hidden; box-shadow: 0 2px 12px rgba(0,0,0,0.1); }
.data-table th { background: #667eea; color: white; padding: 1rem; text-align: left; }
.data-table td { padding: 1rem; border-bottom: 1px solid #e0e0e0; }
.data-table tr:hover { background: #f5f5f5; }
```

### styles/Navbar.css
```css
.navbar { background: white; box-shadow: 0 2px 12px rgba(0,0,0,0.1); }
.nav-container { display: flex; justify-content: space-between; align-items: center; padding: 1rem 2rem; max-width: 1200px; margin: 0 auto; }
.nav-brand { font-size: 1.3rem; font-weight: bold; color: #667eea; }
.nav-menu { display: flex; gap: 1rem; align-items: center; }
.nav-menu button { background: none; border: none; cursor: pointer; padding: 0.5rem 1rem; color: #333; font-weight: 500; transition: color 0.3s; }
.nav-menu button:hover { color: #667eea; }
.btn-logout { background: #e74c3c; color: white; border-radius: 8px; padding: 0.5rem 1rem; }
.btn-logout:hover { background: #c0392b; }
```

## Quick Implementation

Create these directories and copy the code:

```bash
mkdir -p src/{lib,pages,components,styles}
```

Then create each file with the code above.

**For full detailed code of all pages with complete functionality, see DETAILED-CODE.md**
