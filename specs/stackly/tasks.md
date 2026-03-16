# خطة التنفيذ - منصة Stackly

## نظرة عامة

هذه الوثيقة تحتوي على خطة التنفيذ التفصيلية لمنصة Stackly، مقسمة إلى مراحل ومهام قابلة للتنفيذ.

**استراتيجية التنفيذ:** نبدأ بـ Supabase للوصول السريع إلى MVP، ثم ننتقل تدريجياً إلى PostgreSQL + Prisma.

---

# المرحلة 1: MVP مع Supabase (الأشهر 1-3)

## الهدف
بناء MVP وظيفي يسمح للمستخدمين بـ:
- التسجيل وتسجيل الدخول
- إنشاء مساحات عمل ومشاريع
- اختيار قالب بسيط
- معاينة المشروع
- نشر موقع ثابت بسيط

---

## 1. إعداد البيئة والبنية الأساسية

### 1.1 إعداد Monorepo
- [ ] 1.1.1 تثبيت Node.js 24 LTS
- [ ] 1.1.2 تثبيت pnpm
- [ ] 1.1.3 إنشاء هيكل Monorepo
  ```bash
  mkdir stackly && cd stackly
  pnpm init
  ```
- [ ] 1.1.4 إعداد pnpm-workspace.yaml
  ```yaml
  packages:
    - 'apps/*'
    - 'packages/*'
  ```
- [ ] 1.1.5 تثبيت Turborepo
  ```bash
  pnpm add -Dw turbo
  ```
- [ ] 1.1.6 إنشاء turbo.json
- [ ] 1.1.7 إنشاء هيكل المجلدات
  ```
  apps/
    dashboard/
  packages/
    ui/
    supabase/
    config/
  ```

### 1.2 إعداد Supabase Project
- [ ] 1.2.1 إنشاء حساب Supabase (إذا لم يكن موجوداً)
- [ ] 1.2.2 إنشاء مشروع جديد على Supabase Cloud
- [ ] 1.2.3 تثبيت Supabase CLI
  ```bash
  npm install -g supabase
  ```
- [ ] 1.2.4 تسجيل الدخول إلى Supabase CLI
  ```bash
  supabase login
  ```
- [ ] 1.2.5 تهيئة Supabase في المشروع
  ```bash
  supabase init
  ```
- [ ] 1.2.6 ربط المشروع المحلي بـ Supabase Cloud
  ```bash
  supabase link --project-ref <project-ref>
  ```
- [ ] 1.2.7 إنشاء ملف .env.local مع متغيرات Supabase
  ```
  NEXT_PUBLIC_SUPABASE_URL=
  NEXT_PUBLIC_SUPABASE_ANON_KEY=
  SUPABASE_SERVICE_ROLE_KEY=
  ```

### 1.3 إعداد Git و GitHub
- [ ] 1.3.1 تهيئة Git repository
  ```bash
  git init
  ```
- [ ] 1.3.2 إنشاء .gitignore
- [ ] 1.3.3 إنشاء repository على GitHub
- [ ] 1.3.4 ربط المشروع المحلي بـ GitHub
- [ ] 1.3.5 أول commit و push

---

## 2. قاعدة البيانات (Supabase Database)

### 2.1 إنشاء Schema الأساسي
- [ ] 2.1.1 إنشاء migration للجداول الأساسية
  ```bash
  supabase migration new initial_schema
  ```
- [ ] 2.1.2 كتابة SQL للجداول:
  - [ ] workspaces
  - [ ] workspace_members
  - [ ] projects
  - [ ] environments
  - [ ] templates
  - [ ] blocks
  - [ ] components
  - [ ] plans
- [ ] 2.1.3 تطبيق Migration محلياً
  ```bash
  supabase db reset
  ```
- [ ] 2.1.4 مراجعة الجداول في Supabase Studio

### 2.2 إعداد Row Level Security (RLS)
- [ ] 2.2.1 تفعيل RLS على جميع الجداول
- [ ] 2.2.2 إنشاء policies للـ workspaces
  - [ ] SELECT policy
  - [ ] INSERT policy
  - [ ] UPDATE policy
  - [ ] DELETE policy
- [ ] 2.2.3 إنشاء policies للـ projects
- [ ] 2.2.4 إنشاء policies للـ workspace_members
- [ ] 2.2.5 اختبار RLS policies

### 2.3 إنشاء Functions و Triggers
- [ ] 2.3.1 Function لتحديث updated_at تلقائياً
- [ ] 2.3.2 Trigger على workspaces
- [ ] 2.3.3 Trigger على projects
- [ ] 2.3.4 Function للتحقق من صلاحيات المستخدم

### 2.4 Seed Data (بيانات أولية)
- [ ] 2.4.1 إنشاء ملف seed.sql
- [ ] 2.4.2 إضافة خطط الاشتراك (Starter, Professional, Enterprise)
- [ ] 2.4.3 إضافة 3 قوالب أساسية
  - [ ] Landing Page Template
  - [ ] Portfolio Template
  - [ ] Coffee Shop Template
- [ ] 2.4.4 إضافة blocks أساسية (Hero, Features, Footer)
- [ ] 2.4.5 إضافة components أساسية (Button, Card, Input)
- [ ] 2.4.6 تطبيق Seed data
  ```bash
  supabase db seed
  ```

---

## 3. Dashboard Application (Next.js)

### 3.1 إنشاء Next.js App
- [ ] 3.1.1 إنشاء تطبيق Next.js في apps/dashboard
  ```bash
  cd apps
  pnpm create next-app dashboard --typescript --tailwind --app --use-pnpm
  ```
- [ ] 3.1.2 تثبيت التبعيات الأساسية
  ```bash
  pnpm add @supabase/ssr @supabase/supabase-js
  pnpm add -D @types/node
  ```
- [ ] 3.1.3 إعداد TypeScript config
- [ ] 3.1.4 إعداد Tailwind config
- [ ] 3.1.5 اختبار التشغيل
  ```bash
  pnpm dev
  ```

### 3.2 إعداد Supabase Client
- [ ] 3.2.1 إنشاء lib/supabase/client.ts (Client-side)
- [ ] 3.2.2 إنشاء lib/supabase/server.ts (Server-side)
- [ ] 3.2.3 إنشاء lib/supabase/middleware.ts
- [ ] 3.2.4 إعداد middleware.ts في الجذر
- [ ] 3.2.5 اختبار الاتصال بـ Supabase

### 3.3 إعداد UI Library (shadcn/ui)
- [ ] 3.3.1 تهيئة shadcn/ui
  ```bash
  pnpm dlx shadcn-ui@latest init
  ```
- [ ] 3.3.2 إضافة المكونات الأساسية:
  - [ ] Button
  - [ ] Input
  - [ ] Card
  - [ ] Dialog
  - [ ] Form
  - [ ] Select
  - [ ] Toast
  - [ ] Avatar
  - [ ] Dropdown Menu
- [ ] 3.3.3 إنشاء theme config
- [ ] 3.3.4 إعداد الألوان والخطوط

### 3.4 بناء صفحات Authentication
- [ ] 3.4.1 إنشاء layout للـ auth
  ```
  app/(auth)/layout.tsx
  ```
- [ ] 3.4.2 بناء صفحة Login
  - [ ] UI للصفحة
  - [ ] Form validation (zod)
  - [ ] دالة تسجيل الدخول
  - [ ] معالجة الأخطاء
  - [ ] Redirect بعد النجاح
- [ ] 3.4.3 بناء صفحة Register
  - [ ] UI للصفحة
  - [ ] Form validation
  - [ ] دالة التسجيل
  - [ ] إرسال بريد التحقق
  - [ ] معالجة الأخطاء
- [ ] 3.4.4 بناء صفحة Forgot Password
- [ ] 3.4.5 بناء صفحة Reset Password
- [ ] 3.4.6 إضافة Social Login (Google, GitHub)

### 3.5 بناء Dashboard Layout
- [ ] 3.5.1 إنشاء layout للـ dashboard
  ```
  app/(dashboard)/layout.tsx
  ```
- [ ] 3.5.2 بناء Sidebar
  - [ ] Navigation links
  - [ ] User menu
  - [ ] Workspace switcher
- [ ] 3.5.3 بناء Header
  - [ ] Breadcrumbs
  - [ ] Search
  - [ ] Notifications
  - [ ] User avatar
- [ ] 3.5.4 إضافة Protected Route middleware
- [ ] 3.5.5 إضافة Loading states

### 3.6 بناء صفحة Dashboard الرئيسية
- [ ] 3.6.1 إنشاء app/(dashboard)/page.tsx
- [ ] 3.6.2 عرض إحصائيات سريعة
  - [ ] عدد المشاريع
  - [ ] عدد المشاريع المنشورة
  - [ ] آخر التحديثات
- [ ] 3.6.3 عرض المشاريع الأخيرة
- [ ] 3.6.4 Quick actions (إنشاء مشروع جديد)

---

## 4. إدارة Workspaces

### 4.1 صفحة Workspaces
- [ ] 4.1.1 إنشاء app/(dashboard)/workspaces/page.tsx
- [ ] 4.1.2 عرض قائمة Workspaces
- [ ] 4.1.3 زر إنشاء workspace جديد
- [ ] 4.1.4 بطاقة لكل workspace (اسم، عدد المشاريع، الأعضاء)

### 4.2 إنشاء Workspace
- [ ] 4.2.1 Dialog لإنشاء workspace
- [ ] 4.2.2 Form (اسم، slug)
- [ ] 4.2.3 Validation
- [ ] 4.2.4 API call لإنشاء workspace
- [ ] 4.2.5 إضافة المستخدم كـ owner تلقائياً
- [ ] 4.2.6 Redirect إلى workspace الجديد

### 4.3 إعدادات Workspace
- [ ] 4.3.1 إنشاء app/(dashboard)/workspaces/[id]/settings/page.tsx
- [ ] 4.3.2 تبويبات (General, Members, Billing)
- [ ] 4.3.3 تبويب General:
  - [ ] تعديل الاسم
  - [ ] تعديل الصورة
  - [ ] حذف workspace
- [ ] 4.3.4 تبويب Members:
  - [ ] عرض الأعضاء
  - [ ] دعوة عضو جديد
  - [ ] تغيير الدور
  - [ ] إزالة عضو
- [ ] 4.3.5 تبويب Billing:
  - [ ] عرض الخطة الحالية
  - [ ] الترقية/التخفيض
  - [ ] تاريخ الفواتير

---

## 5. إدارة Projects

### 5.1 صفحة Projects
- [ ] 5.1.1 إنشاء app/(dashboard)/projects/page.tsx
- [ ] 5.1.2 عرض قائمة Projects
- [ ] 5.1.3 Filters (الحالة، القالب، التاريخ)
- [ ] 5.1.4 Search
- [ ] 5.1.5 Grid/List view toggle
- [ ] 5.1.6 زر إنشاء project جديد

### 5.2 إنشاء Project - اختيار القالب
- [ ] 5.2.1 Dialog/Page لإنشاء project
- [ ] 5.2.2 الخطوة 1: اختيار workspace
- [ ] 5.2.3 الخطوة 2: عرض القوالب المتاحة
  - [ ] Fetch templates من Supabase
  - [ ] عرض بطاقات القوالب (صورة، اسم، وصف)
  - [ ] Filter حسب الفئة
- [ ] 5.2.4 الخطوة 3: معاينة القالب
  - [ ] عرض تفاصيل القالب
  - [ ] معاينة مصغرة
  - [ ] زر "استخدام هذا القالب"
- [ ] 5.2.5 الخطوة 4: تفاصيل المشروع
  - [ ] اسم المشروع
  - [ ] slug
  - [ ] وصف (اختياري)

### 5.3 إنشاء Project - API
- [ ] 5.3.1 Server Action لإنشاء project
- [ ] 5.3.2 Validation
- [ ] 5.3.3 التحقق من حدود الخطة
- [ ] 5.3.4 إنشاء project في Supabase
- [ ] 5.3.5 إنشاء environment افتراضي (production)
- [ ] 5.3.6 نسخ config من القالب
- [ ] 5.3.7 Redirect إلى صفحة المشروع

### 5.4 صفحة Project Details
- [ ] 5.4.1 إنشاء app/(dashboard)/projects/[id]/page.tsx
- [ ] 5.4.2 عرض معلومات المشروع
  - [ ] الاسم والوصف
  - [ ] الحالة (draft, deployed, etc)
  - [ ] URL (إذا كان منشور)
  - [ ] آخر تحديث
- [ ] 5.4.3 Quick actions:
  - [ ] فتح المحرر
  - [ ] معاينة
  - [ ] نشر
  - [ ] إعدادات
- [ ] 5.4.4 عرض Environments
- [ ] 5.4.5 عرض آخر Deployments

### 5.5 إعدادات Project
- [ ] 5.5.1 إنشاء app/(dashboard)/projects/[id]/settings/page.tsx
- [ ] 5.5.2 تبويب General:
  - [ ] تعديل الاسم
  - [ ] تعديل الوصف
  - [ ] تغيير القالب (إذا أمكن)
  - [ ] حذف المشروع
- [ ] 5.5.3 تبويب Domain:
  - [ ] عرض الدومين الافتراضي
  - [ ] إضافة دومين مخصص
  - [ ] إعدادات DNS
- [ ] 5.5.4 تبويب Environment Variables
- [ ] 5.5.5 تبويب Danger Zone (حذف، أرشفة)

---

## 6. المحرر الأساسي (Basic Editor)

### 6.1 إعداد Editor Page
- [ ] 6.1.1 إنشاء app/(editor)/projects/[id]/editor/page.tsx
- [ ] 6.1.2 Layout مخصص للمحرر (full screen)
- [ ] 6.1.3 تحميل project config من Supabase
- [ ] 6.1.4 تحميل template schema

### 6.2 Editor UI Structure
- [ ] 6.2.1 بناء Toolbar (أعلى)
  - [ ] اسم المشروع
  - [ ] زر Save
  - [ ] زر Preview
  - [ ] زر Publish
  - [ ] Undo/Redo
- [ ] 6.2.2 بناء Left Sidebar (Components Panel)
  - [ ] تبويبات (Blocks, Components, Assets)
  - [ ] عرض Blocks المتاحة
  - [ ] Search/Filter
- [ ] 6.2.3 بناء Canvas (المنتصف)
  - [ ] عرض الصفحة الحالية
  - [ ] Responsive preview (Desktop, Tablet, Mobile)
- [ ] 6.2.4 بناء Right Sidebar (Properties Panel)
  - [ ] عرض خصائص العنصر المحدد
  - [ ] Form لتعديل الخصائص
- [ ] 6.2.5 بناء Bottom Panel (Layers - اختياري)

### 6.3 Canvas Implementation (مبسط)
- [ ] 6.3.1 عرض sections من template config
- [ ] 6.3.2 عرض blocks داخل كل section
- [ ] 6.3.3 تمييز العنصر المحدد
- [ ] 6.3.4 Click handler لاختيار عنصر
- [ ] 6.3.5 عرض placeholder للـ slots الفارغة

### 6.4 Properties Panel
- [ ] 6.4.1 عرض خصائص العنصر المحدد
- [ ] 6.4.2 Text inputs للنصوص
- [ ] 6.4.3 Color picker للألوان
- [ ] 6.4.4 Image uploader للصور
- [ ] 6.4.5 تحديث config عند التغيير
- [ ] 6.4.6 Debounced save

### 6.5 State Management (Zustand)
- [ ] 6.5.1 إنشاء editor store
- [ ] 6.5.2 State:
  - [ ] project
  - [ ] template
  - [ ] config (current)
  - [ ] selectedElement
  - [ ] history (undo/redo)
- [ ] 6.5.3 Actions:
  - [ ] loadProject
  - [ ] updateElement
  - [ ] selectElement
  - [ ] undo/redo
  - [ ] save

### 6.6 Save Functionality
- [ ] 6.6.1 Debounced auto-save (كل 3 ثواني)
- [ ] 6.6.2 Manual save button
- [ ] 6.6.3 تحديث project config في Supabase
- [ ] 6.6.4 عرض حالة الحفظ (Saving, Saved, Error)
- [ ] 6.6.5 معالجة الأخطاء

---

## 7. Preview & Deploy (مبسط)

### 7.1 Preview Functionality
- [ ] 7.1.1 زر Preview في المحرر
- [ ] 7.1.2 فتح Preview في نافذة/تبويب جديد
- [ ] 7.1.3 إنشاء صفحة preview
  ```
  app/preview/[projectId]/page.tsx
  ```
- [ ] 7.1.4 تحميل project config
- [ ] 7.1.5 Render الصفحة من config
- [ ] 7.1.6 تطبيق الـ theme (colors, fonts)

### 7.2 Static Site Generation (مبسط)
- [ ] 7.2.1 إنشاء Supabase Edge Function للـ build
  ```
  supabase/functions/build-project/index.ts
  ```
- [ ] 7.2.2 تحميل project config
- [ ] 7.2.3 توليد HTML من config
- [ ] 7.2.4 توليد CSS من theme
- [ ] 7.2.5 حفظ الملفات في Supabase Storage
- [ ] 7.2.6 إرجاع URL للموقع

### 7.3 Deploy Functionality
- [ ] 7.3.1 زر Deploy في المحرر
- [ ] 7.3.2 Dialog للتأكيد
- [ ] 7.3.3 استدعاء Edge Function للـ build
- [ ] 7.3.4 عرض progress/loading
- [ ] 7.3.5 تحديث project status إلى "deployed"
- [ ] 7.3.6 حفظ deployment record
- [ ] 7.3.7 عرض رسالة نجاح مع URL

### 7.4 Supabase Storage Setup
- [ ] 7.4.1 إنشاء bucket للمشاريع المنشورة
  ```
  deployed-sites
  ```
- [ ] 7.4.2 إعداد policies للوصول العام
- [ ] 7.4.3 إعداد CDN (Supabase CDN)

---

## 8. Templates & Registry (أساسي)

### 8.1 Template Viewer
- [ ] 8.1.1 صفحة لعرض جميع القوالب
  ```
  app/(dashboard)/templates/page.tsx
  ```
- [ ] 8.1.2 Grid view للقوالب
- [ ] 8.1.3 Filter حسب الفئة
- [ ] 8.1.4 Search
- [ ] 8.1.5 صفحة تفاصيل القالب
  ```
  app/(dashboard)/templates/[id]/page.tsx
  ```

### 8.2 إنشاء القوالب الأولية
- [ ] 8.2.1 Landing Page Template:
  - [ ] Hero section
  - [ ] Features section
  - [ ] CTA section
  - [ ] Footer
- [ ] 8.2.2 Portfolio Template:
  - [ ] Hero with image
  - [ ] Projects grid
  - [ ] About section
  - [ ] Contact form
- [ ] 8.2.3 Coffee Shop Template:
  - [ ] Hero with menu
  - [ ] Products grid
  - [ ] Location/Hours
  - [ ] Footer

### 8.3 Template Schema
- [ ] 8.3.1 تعريف JSON schema للقوالب
- [ ] 8.3.2 Validation للـ schema
- [ ] 8.3.3 إضافة القوالب إلى Supabase

---

## 9. Testing & Quality Assurance

### 9.1 Unit Tests
- [ ] 9.1.1 إعداد Vitest
- [ ] 9.1.2 Tests للـ Supabase utilities
- [ ] 9.1.3 Tests للـ Editor store
- [ ] 9.1.4 Tests للـ validation functions

### 9.2 Integration Tests
- [ ] 9.2.1 إعداد Playwright
- [ ] 9.2.2 Tests للـ Auth flow
- [ ] 9.2.3 Tests لإنشاء workspace
- [ ] 9.2.4 Tests لإنشاء project
- [ ] 9.2.5 Tests للمحرر الأساسي

### 9.3 Manual Testing
- [ ] 9.3.1 اختبار كامل للـ user flow
- [ ] 9.3.2 اختبار على متصفحات مختلفة
- [ ] 9.3.3 اختبار responsive design
- [ ] 9.3.4 اختبار الأداء
- [ ] 9.3.5 اختبار الأمان (RLS)

---

## 10. Deployment & Launch

### 10.1 إعداد Production Environment
- [ ] 10.1.1 إنشاء Supabase project للـ production
- [ ] 10.1.2 تطبيق migrations على production
- [ ] 10.1.3 تطبيق seed data
- [ ] 10.1.4 إعداد environment variables

### 10.2 Deploy Dashboard
- [ ] 10.2.1 اختيار hosting provider (Vercel/Netlify)
- [ ] 10.2.2 ربط GitHub repository
- [ ] 10.2.3 إعداد build settings
- [ ] 10.2.4 إعداد environment variables
- [ ] 10.2.5 Deploy أول نسخة
- [ ] 10.2.6 اختبار production deployment

### 10.3 Domain & SSL
- [ ] 10.3.1 شراء domain (stackly.app)
- [ ] 10.3.2 إعداد DNS
- [ ] 10.3.3 إعداد SSL certificate
- [ ] 10.3.4 اختبار الدومين

### 10.4 Monitoring & Analytics
- [ ] 10.4.1 إعداد error tracking (Sentry)
- [ ] 10.4.2 إعداد analytics (Plausible/Umami)
- [ ] 10.4.3 إعداد uptime monitoring
- [ ] 10.4.4 إعداد performance monitoring

### 10.5 Documentation
- [ ] 10.5.1 كتابة README
- [ ] 10.5.2 كتابة user guide
- [ ] 10.5.3 كتابة API documentation
- [ ] 10.5.4 إنشاء video tutorials

### 10.6 Launch
- [ ] 10.6.1 Soft launch (beta users)
- [ ] 10.6.2 جمع feedback
- [ ] 10.6.3 إصلاح bugs
- [ ] 10.6.4 Public launch
- [ ] 10.6.5 Marketing & promotion

---

# ملاحظات مهمة

## الأولويات
1. **الأساسيات أولاً**: Auth, Workspaces, Projects
2. **المحرر البسيط**: عرض وتعديل أساسي فقط
3. **Deploy مبسط**: Static sites فقط في البداية
4. **التحسينات لاحقاً**: Drag & Drop, Realtime, Advanced features

## التقديرات الزمنية
- **الأسبوع 1-2**: إعداد البيئة + قاعدة البيانات + Auth
- **الأسبوع 3-4**: Dashboard + Workspaces + Projects
- **الأسبوع 5-7**: المحرر الأساسي
- **الأسبوع 8-9**: Preview & Deploy
- **الأسبوع 10-11**: Templates + Testing
- **الأسبوع 12**: Deployment + Launch

## نصائح للتنفيذ
1. ابدأ بأبسط نسخة ممكنة (MVP حقيقي)
2. اختبر كل ميزة قبل الانتقال للتالية
3. استخدم Supabase Studio لمراقبة البيانات
4. commit بشكل متكرر
5. اطلب مراجعة الكود إذا أمكن

---

**تاريخ الإنشاء:** 2026-03-15  
**الإصدار:** 1.0.0  
**الحالة:** جاهز للتنفيذ
