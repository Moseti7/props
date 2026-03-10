-- THINKDIFFERENT VIRTUAL SCHOOL DATABASE SCHEMA
-- SUPABASE POSTGRESQL COMPLETE SCRIPT

-- =========================
-- EXTENSIONS
-- =========================
create extension if not exists "uuid-ossp";

-- =========================
-- USER ROLES TABLE
-- =========================
create table roles (
    id uuid primary key default uuid_generate_v4(),
    role_name text unique not null
);

insert into roles (role_name) values
('manager'),
('teacher'),
('student');

-- =========================
-- USERS TABLE (LINKED TO SUPABASE AUTH)
-- =========================
create table users (
    id uuid primary key references auth.users(id) on delete cascade,
    full_name text,
    email text,
    role_id uuid references roles(id),
    created_at timestamp default now()
);

-- =========================
-- STUDENTS TABLE
-- =========================
create table students (
    id uuid primary key default uuid_generate_v4(),
    user_id uuid references users(id),
    registration_number text unique,
    curriculum text check (curriculum in ('CBC','844')),
    full_name text not null,
    gender text,
    phone text,
    email text,
    date_of_birth date,
    physical_location text,
    created_at timestamp default now()
);

-- =========================
-- PARENT / GUARDIAN
-- =========================
create table guardians (
    id uuid primary key default uuid_generate_v4(),
    student_id uuid references students(id) on delete cascade,
    name text,
    phone text,
    relationship text
);

-- =========================
-- TEACHERS TABLE
-- =========================
create table teachers (
    id uuid primary key default uuid_generate_v4(),
    user_id uuid references users(id),
    full_name text,
    gender text,
    phone text,
    email text,
    id_number text,
    subject_specialization text,
    description text,
    verified boolean default false,
    created_at timestamp default now()
);

-- =========================
-- TEACHER DOCUMENT UPLOADS
-- =========================
create table teacher_documents (
    id uuid primary key default uuid_generate_v4(),
    teacher_id uuid references teachers(id) on delete cascade,
    document_url text,
    uploaded_at timestamp default now()
);

-- =========================
-- SUBJECTS TABLE
-- =========================
create table subjects (
    id uuid primary key default uuid_generate_v4(),
    subject_code text,
    subject_name text,
    category text
);

insert into subjects (subject_code,subject_name,category) values
('101','English','KCSE'),
('102','Kiswahili','KCSE'),
('121','Mathematics','KCSE'),
('122','Mathematics Alternative B','KCSE'),
('231','Biology','KCSE'),
('232','Physics','KCSE'),
('233','Chemistry','KCSE'),
('237','General Science','KCSE'),
('311','History and Government','KCSE'),
('312','Geography','KCSE'),
('313','CRE','KCSE'),
('314','IRE','KCSE'),
('315','HRE','KCSE'),
('565','Business Studies','KCSE');

-- =========================
-- STUDENT SUBJECT SELECTION
-- =========================
create table student_subjects (
    id uuid primary key default uuid_generate_v4(),
    student_id uuid references students(id) on delete cascade,
    subject_id uuid references subjects(id)
);

-- =========================
-- PAYMENTS TABLE
-- =========================
create table payments (
    id uuid primary key default uuid_generate_v4(),
    student_id uuid references students(id),
    amount numeric,
    payment_method text default 'mpesa',
    mpesa_receipt text,
    payment_date timestamp default now()
);

-- =========================
-- TEACHER PAYMENTS
-- =========================
create table teacher_payments (
    id uuid primary key default uuid_generate_v4(),
    teacher_id uuid references teachers(id),
    hours_worked numeric,
    rate numeric default 500,
    total_amount numeric,
    paid_at timestamp default now()
);

-- =========================
-- EXAM REGISTRATION
-- =========================
create table exam_registrations (
    id uuid primary key default uuid_generate_v4(),
    student_id uuid references students(id),
    exam_type text,
    kcpe_index text,
    kcpe_year text,
    assessment_number text,
    passport_photo text,
    submitted_at timestamp default now(),
    status text default 'pending'
);

-- =========================
-- DOCUMENTS FOR EXAM BOOKING
-- =========================
create table exam_documents (
    id uuid primary key default uuid_generate_v4(),
    exam_registration_id uuid references exam_registrations(id) on delete cascade,
    document_url text,
    uploaded_at timestamp default now()
);

-- =========================
-- MEET THE TEAM
-- =========================
create table team_members (
    id uuid primary key default uuid_generate_v4(),
    name text,
    title text,
    description text
);

insert into team_members (name,title,description) values
('Mr. Jesse Wainaina','Senior Adult Education Instructor','Specialized in KCSE preparation'),
('Mr. John Ombasa','Adult Education Instructor','Systems management, timetables, exams and finance');

-- =========================
-- CAREER PATHWAYS
-- =========================
create table career_pathways (
    id uuid primary key default uuid_generate_v4(),
    course_name text,
    key_subjects text,
    description text,
    universities text
);

-- =========================
-- ADMISSIONS TABLE
-- =========================
create table admissions (
    id uuid primary key default uuid_generate_v4(),
    student_id uuid references students(id),
    admission_date timestamp default now(),
    status text default 'pending'
);

-- =========================
-- AUTO REGISTRATION NUMBER FUNCTION
-- =========================
create or replace function generate_student_number()
returns trigger as $$
begin

if NEW.curriculum = 'CBC' then
NEW.registration_number =
'TD/CBA/' || lpad((select count(*)+1 from students where curriculum='CBC')::text,3,'0');

else
NEW.registration_number =
'TD/HS/' || lpad((select count(*)+1 from students where curriculum='844')::text,3,'0');

end if;

return NEW;

end;
$$ language plpgsql;

create trigger student_number_trigger
before insert on students
for each row
execute procedure generate_student_number();

-- =========================
-- ENABLE ROW LEVEL SECURITY
-- =========================
alter table users enable row level security;
alter table students enable row level security;
alter table teachers enable row level security;
alter table payments enable row level security;
alter table student_subjects enable row level security;
alter table exam_registrations enable row level security;

-- =========================
-- RLS POLICIES
-- =========================

-- STUDENTS CAN VIEW THEIR DATA
create policy "Students view own data"
on students
for select
using (auth.uid() = user_id);

-- STUDENTS INSERT OWN DATA
create policy "Students insert"
on students
for insert
with check (auth.uid() = user_id);

-- TEACHERS VIEW OWN DATA
create policy "Teacher view own"
on teachers
for select
using (auth.uid() = user_id);

-- MANAGER FULL ACCESS
create policy "Manager full access"
on students
for all
using (
exists(
select 1 from users
where users.id = auth.uid()
and users.role_id = (select id from roles where role_name='manager')
)
);

-- =========================
-- PERMISSIONS
-- =========================

grant select on students to authenticated;
grant insert on students to authenticated;

grant select on teachers to authenticated;
grant insert on teachers to authenticated;

grant select on subjects to authenticated;

grant select on career_pathways to authenticated;

grant select on team_members to authenticated;

-- =========================
-- END OF SCRIPT
-- =========================# props
Entire structure of my application and database needs
