# Complete Setup Guide - Academic Portal

## Quick Start (5 minutes)

### 1. Create Vite React Project

```bash
npm create vite@latest academic-portal -- --template react
cd academic-portal
npm install @supabase/supabase-js react-router-dom lucide-react
```

### 2. Create `.env.local`

```
VITE_SUPABASE_URL=https://your-project.supabase.co
VITE_SUPABASE_ANON_KEY=your_anon_key_here
```

### 3. Run SQL Schema in Supabase SQL Editor

Copy-paste this entire SQL block:

```sql
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

CREATE TABLE profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  role TEXT CHECK (role IN ('admin', 'student')) DEFAULT 'student',
  full_name TEXT,
  email TEXT,
  class TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE courses (
  id BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
  title TEXT NOT NULL,
  code TEXT UNIQUE NOT NULL,
  description TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE assessments (
  id BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
  course_id BIGINT NOT NULL REFERENCES courses(id) ON DELETE CASCADE,
  title TEXT NOT NULL,
  type TEXT CHECK (type IN ('quiz', 'assignment', 'exam')),
  description TEXT,
  due_date DATE,
  total_marks NUMERIC,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE student_assessments (
  id BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
  student_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  assessment_id BIGINT NOT NULL REFERENCES assessments(id) ON DELETE CASCADE,
  status TEXT DEFAULT 'assigned',
  submission_date TIMESTAMP,
  marks_obtained NUMERIC,
  feedback TEXT,
  created_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(student_id, assessment_id)
);

ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE courses ENABLE ROW LEVEL SECURITY;
ALTER TABLE assessments ENABLE ROW LEVEL SECURITY;
ALTER TABLE student_assessments ENABLE ROW LEVEL SECURITY;

CREATE OR REPLACE FUNCTION is_admin()
RETURNS BOOLEAN AS $$
BEGIN
  RETURN (SELECT role = 'admin' FROM profiles WHERE id = auth.uid());
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

CREATE POLICY "Users view own profile" ON profiles FOR SELECT USING (auth.uid() = id);
CREATE POLICY "Admin view all profiles" ON profiles FOR SELECT USING (is_admin());
CREATE POLICY "Admin manage profiles" ON profiles FOR ALL USING (is_admin());

CREATE POLICY "All authenticated view courses" ON courses FOR SELECT TO authenticated USING (true);
CREATE POLICY "Admin manage courses" ON courses FOR ALL USING (is_admin());

CREATE POLICY "All authenticated view assessments" ON assessments FOR SELECT TO authenticated USING (true);
CREATE POLICY "Admin manage assessments" ON assessments FOR ALL USING (is_admin());

CREATE POLICY "Students view own assessments" ON student_assessments FOR SELECT USING (student_id = auth.uid());
CREATE POLICY "Admin manage all student assessments" ON student_assessments FOR ALL USING (is_admin());

CREATE OR REPLACE FUNCTION create_profile_on_signup()
RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO profiles (id, email, role, full_name)
  VALUES (NEW.id, NEW.email, 'student', SPLIT_PART(NEW.email, '@', 1));
  RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

CREATE TRIGGER on_auth_user_created
  AFTER INSERT ON auth.users
  FOR EACH ROW EXECUTE FUNCTION create_profile_on_signup();
```

### 4. Create Admin Account

1. Sign up via the app login page with: admin@example.com / admin123
2. In Supabase dashboard, go to profiles table
3. Update the admin@example.com profile: set role = 'admin'

### 5. Run the App

```bash
npm run dev
```

Open http://localhost:5173

---

## File Structure to Create

Create these files in your src directory:

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
│   ├── PrivateRoute.jsx
│   └── LoadingSpinner.jsx
├── styles/
│   ├── Auth.css
│   ├── Dashboard.css
│   └── Management.css
├── App.jsx
└── index.css
```

---

## Code Files

### src/lib/supabase.js

```javascript
import { createClient } from '@supabase/supabase-js';

const supabaseUrl = import.meta.env.VITE_SUPABASE_URL;
const supabaseKey = import.meta.env.VITE_SUPABASE_ANON_KEY;

export const supabase = createClient(supabaseUrl, supabaseKey);

export const getCurrentUser = async () => {
  const { data: { user } } = await supabase.auth.getUser();
  return user;
};

export const getUserProfile = async (userId) => {
  const { data, error } = await supabase
    .from('profiles')
    .select('*')
    .eq('id', userId)
    .single();
  return { data, error };
};

export const signOut = async () => {
  return await supabase.auth.signOut();
};
```

**See FULL-CODE.md for all remaining files**
