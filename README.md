# Smart Bookmark App

A modern bookmark manager built with Next.js App Router, Supabase, and Tailwind CSS.

## Features

- 🔐 Google OAuth Authentication
- 📚 Add bookmarks with title and URL
- 🔒 Private bookmarks per user
- 🔄 Real-time updates across tabs
- 🗑️ Delete bookmarks
- 🎨 Clean, responsive UI with Tailwind CSS

## Tech Stack

- **Frontend**: Next.js 14 (App Router)
- **Authentication**: Supabase Auth (Google OAuth)
- **Database**: Supabase (PostgreSQL)
- **Real-time**: Supabase Realtime
- **Styling**: Tailwind CSS
- **Deployment**: Vercel

## Live Demo

[Add your Vercel URL here after deployment]

## Local Development

1. Clone the repository
   ```bash
   git clone https://github.com/YOUR_USERNAME/smart-bookmark-app.git
   cd smart-bookmark-app
   ```

2. Install dependencies
   ```bash
   npm install
   ```

3. Create `.env.local` file and add your Supabase credentials:
   ```env
   NEXT_PUBLIC_SUPABASE_URL=your_supabase_project_url
   NEXT_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key
   ```

4. Run the development server
   ```bash
   npm run dev
   ```

5. Open [http://localhost:3000](http://localhost:3000)

## Database Setup

Run this SQL in your Supabase SQL editor:

```sql
-- Create bookmarks table
CREATE TABLE bookmarks (
    id BIGSERIAL PRIMARY KEY,
    user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
    title TEXT NOT NULL,
    url TEXT NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT TIMEZONE('utc'::text, NOW()) NOT NULL,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT TIMEZONE('utc'::text, NOW()) NOT NULL
);

-- Enable Row Level Security
ALTER TABLE bookmarks ENABLE ROW LEVEL SECURITY;

-- Create policies
CREATE POLICY "Users can view their own bookmarks" 
    ON bookmarks FOR SELECT 
    USING (auth.uid() = user_id);

CREATE POLICY "Users can insert their own bookmarks" 
    ON bookmarks FOR INSERT 
    WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can update their own bookmarks" 
    ON bookmarks FOR UPDATE 
    USING (auth.uid() = user_id);

CREATE POLICY "Users can delete their own bookmarks" 
    ON bookmarks FOR DELETE 
    USING (auth.uid() = user_id);

-- Enable realtime
ALTER PUBLICATION supabase_realtime ADD TABLE bookmarks;

-- Create updated_at trigger
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ language 'plpgsql';

CREATE TRIGGER update_bookmarks_updated_at 
    BEFORE UPDATE ON bookmarks 
    FOR EACH ROW 
    EXECUTE FUNCTION update_updated_at_column();
```

## Challenges Faced & Solutions

### 1. Real-time Updates Across Tabs
- **Problem**: Implementing real-time updates that work across different browser tabs
- **Solution**: Used Supabase Realtime subscriptions with proper channel management and cleanup in useEffect

### 2. Authentication Flow with App Router
- **Problem**: Setting up Google OAuth callback handling with Next.js App Router
- **Solution**: Created a dedicated API route for auth callback and used middleware for route protection

### 3. Server Actions with Supabase
- **Problem**: Integrating Supabase with Next.js Server Actions while maintaining security
- **Solution**: Used server-side Supabase client in actions and validated user authentication before operations

## Deployment

This app is deployed on Vercel. To deploy your own:

1. Push code to GitHub
2. Import project to Vercel
3. Add environment variables
4. Deploy

## License

MIT
