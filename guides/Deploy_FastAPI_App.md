# Deploying Your FastAPI App to Render.com

Welcome, brave developer! Now we're going to take your shiny new FastAPI app and launch it into the wild world of the internet. Don't worry if this feels overwhelming – we've all been there, staring at deployment docs wondering if we need a computer science degree just to put our "Hello World" app online. Spoiler alert: you don't!

This guide will walk you through every step, from creating a Render account to watching your app live on the internet. Grab some coffee (or tea, we don't judge), and let's turn you into a deployment wizard!

## What is Render.com Anyway?

Think of Render.com as a magical place where your code goes to live its best life on the internet. It's like Airbnb for your applications – they provide the hosting, you provide the awesome code. Best part? They have a generous free tier perfect for student projects and side hustles.

## Prerequisites (The "You'll Need This Stuff" Section)

Before we dive in, make sure you have:

- Your FastAPI app with MongoDB integration (which you already have - nice work!)
- A MongoDB Atlas cluster set up and running (if you haven't done this yet, check out the "Integrate_with_MongoDB.md" guide first)
- Your MongoDB connection string handy (you'll need this for environment variables)
- A GitHub account (because we're going to be professional about this)
- About 45 minutes of your time (a bit longer since we're dealing with a database)
- A decent internet connection
- Patience (deployment can be finicky sometimes, but we'll get through it together)

## Step 1: Prepare Your FastAPI App for the Big World

### 1.1 Create a requirements.txt file

Activate your virtual environment if you haven't:

**On Windows (Command Prompt):**

```cmd
.venv\Scripts\activate
```

**On Windows (PowerShell):**

```powershell
.venv\Scripts\Activate.ps1
```

**On Mac/Linux (Terminal):**

```bash
source .venv/bin/activate
```

Then, we need to tell Render exactly which Python packages your app needs. Since you're using `pyproject.toml`, we'll create a `requirements.txt` file that Render can understand.

**For Windows users:**

```bash
pip freeze > requirements.txt
```

**For Mac users:**

```bash
pip freeze > requirements.txt
```

**Alternative method if you want to be more precise:**
Create a file called `requirements.txt` in your project folder and add:

```
fastapi[standard]>=0.116.1
pymongo>=4.0.0
python-dotenv>=0.19.0
```

**Note:** Since your app integrates with MongoDB, make sure `pymongo` and `python-dotenv` are included. These should already be installed if you followed the MongoDB integration guide!

### 1.2 Create a startup script

Render needs to know how to start your app. Create a file called `start.sh` in your project root:

Content of `start.sh`:

```bash
#!/bin/bash
fastapi run --port $PORT
```

**Important for Windows users:** When you save this file, make sure it doesn't get saved as `start.sh.txt`. Windows likes to add extra extensions when you're not looking!

### 1.3 Make your start script executable (Mac users only)

**For Mac users:**

```bash
chmod +x start.sh
```

**For Windows users:** You can skip this step. Windows handles things differently, and that's okay!

## Step 2: Get Your Code on GitHub

### 2.1 Create a GitHub Repository

1. Go to [GitHub.com](https://github.com) and sign in (or sign up if you haven't already)
2. Click the green "New" button or the "+" icon in the top right corner
3. Name your repository something descriptive like `my-awesome-fastapi-app` (avoid spaces, use hyphens or underscores)
4. Make it public (unless you're planning world domination and need secrecy)
5. Don't initialize with README since you already have files
6. Click "Create repository"

### 2.2 Connect Your Local Code to GitHub

If you haven't already initialized git in your project:

**For both Windows and Mac users:**

```bash
git init
git add .
git commit -m "Initial commit - ready for deployment!"
```

Now connect to your GitHub repository (replace `YOUR_USERNAME` and `YOUR_REPO_NAME`):

```bash
git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPO_NAME.git
git branch -M main
git push -u origin main
```

If this is your first time using git, you might need to set up your identity:

```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

## Step 3: Sign Up for Render.com

### 3.1 Create Your Render Account

1. Head over to [Render.com](https://render.com)
2. Click "Get Started" (it's usually a big, friendly button)
3. You can sign up with GitHub (recommended - it makes connecting your repositories easier)
4. If you choose GitHub, authorize Render to access your repositories
5. Complete your profile setup

### 3.2 Explore the Dashboard

Take a moment to look around. The Render dashboard is pretty intuitive, but it's always good to get your bearings before diving in.

## Step 4: Deploy Your FastAPI App

### 4.1 Create a New Web Service

1. From your Render dashboard, click "New +" button
2. Select "Web Service"
3. Choose "Build and deploy from a Git repository"
4. Connect your GitHub account if you haven't already
5. Find your FastAPI repository and click "Connect"

### 4.2 Configure Your Deployment

Now comes the fun part - telling Render how to run your app:

**Basic Settings:**

- **Name:** Give your service a name (like `my-fastapi-app`)
- **Root Directory:** Leave this blank (unless your app is in a subfolder)
- **Environment:** Select `Python 3`
- **Region:** Choose the one closest to you (or your users)
- **Branch:** `main` (or whatever your default branch is)

**Build Settings:**

- **Build Command:** `pip install -r requirements.txt`
- **Start Command:** `./start.sh` (Mac) or `fastapi run --port $PORT` (Windows, or if the script doesn't work)

**Instance Type:**

- Select "Free" (unless you're feeling generous with your money)

### 4.3 Advanced Settings (Essential for MongoDB)

Click on "Advanced" and add these environment variables:

- **Key:** `PYTHON_VERSION` **Value:** `3.11.0` (or whatever version you're using)
- **Key:** `PORT` **Value:** `10000` (Render uses this port)
- **Key:** `MONGODB_URL` **Value:** `your_mongodb_atlas_connection_string_here`
- **Key:** `DATABASE_NAME` **Value:** `comssa-backend-workshop` (or whatever you named your database)

**Important:** Replace `your_mongodb_atlas_connection_string_here` with your actual MongoDB Atlas connection string from the integration guide. It should look something like:
`mongodb+srv://username:password@cluster0.xyz123.mongodb.net/?retryWrites=true&w=majority`

## Step 4.5: Configure MongoDB Atlas for Deployment

Before we deploy, we need to make sure MongoDB Atlas allows connections from Render.com servers. Don't worry, this is easier than it sounds!

### Update Network Access in MongoDB Atlas

1. **Log into your MongoDB Atlas account** at [cloud.mongodb.com](https://cloud.mongodb.com)

2. **Navigate to Network Access:**

   - Click on "Network Access" in the left sidebar
   - You should see your current IP addresses listed

3. **Add Render.com to allowed IPs:**
   - Click "ADD IP ADDRESS"
   - Select "ALLOW ACCESS FROM ANYWHERE" (this option adds `0.0.0.0/0`)
   - Click "Confirm"

**Wait, is "Allow Access From Anywhere" safe?**
Don't panic! While this wouldn't be recommended for a production application, for the scope of this workshop and learning purposes, this approach is perfectly safe. Your database is still protected by:

- Your username and password (which are in your connection string)
- MongoDB Atlas's built-in security features
- The fact that only applications with the correct connection string can connect

**Note:** In a real production environment, you'd want to be more restrictive with IP access, but for learning and student projects, this is the most straightforward approach.

1. **Wait for the changes to take effect** (usually takes 1-2 minutes)

## Step 5: Deploy and Cross Your Fingers

1. Review all your settings one more time
2. Click "Create Web Service"
3. Watch the build logs (this is oddly satisfying)
4. Wait for the magic to happen (usually takes 2-5 minutes)

## Step 6: Test Your Deployment

Once the deployment is complete, Render will give you a URL that looks something like:
`https://your-app-name.onrender.com`

Click on it! If everything worked correctly, you should see your FastAPI app's response:

```json
{ "message": "Hello World! 🌍" }
```

You can also test your other endpoints:

- `https://your-app-name.onrender.com/hello/YourName`
- `https://your-app-name.onrender.com/info`

### Test Your Database Connection

If your app includes database endpoints (like `/users` for creating/reading users), test those too:

- `https://your-app-name.onrender.com/users` (if you have a get all users endpoint)
- Try creating a user with a POST request to `https://your-app-name.onrender.com/users`

**Pro Tip:** Use the automatic FastAPI docs at `https://your-app-name.onrender.com/docs` to test your API endpoints directly in the browser!

## Troubleshooting (When Things Go Wrong)

Don't panic! Deployment rarely works perfectly on the first try. Here are common issues and fixes:

### Build Failed

- Check your `requirements.txt` file for typos
- Make sure all your dependencies are listed
- Look at the build logs for specific error messages

### App Won't Start

- Verify your start command is correct
- Check that your `main.py` file is in the root directory
- Ensure the port configuration is correct

### 404 Errors

- Make sure your app is actually running (check the logs)
- Verify your endpoints are correctly defined
- Try the root URL first (`/`)

### "My App is Slow to Wake Up"

This is normal for free tier services! Render puts your app to sleep after 15 minutes of inactivity. The first request after sleeping takes 30-60 seconds to wake up. This is the price of free hosting, but hey, it's free!

### Database Connection Issues

If your app starts but database operations fail, check these common culprits:

**MongoDB Atlas Network Access:**

- Make sure you've allowed access from anywhere (`0.0.0.0/0`) in MongoDB Atlas Network Access settings
- Wait 1-2 minutes after changing network settings for them to take effect

**Environment Variables:**

- Double-check that your `MONGODB_URL` environment variable is set correctly in Render
- Ensure your connection string includes the username and password
- Verify your `DATABASE_NAME` environment variable matches your MongoDB database name

**Connection String Format:**

- Your connection string should look like: `mongodb+srv://username:password@cluster0.xyz123.mongodb.net/?retryWrites=true&w=majority`
- Make sure there are no extra spaces or characters

**Check the Logs:**

- Go to your Render dashboard and check the logs for specific error messages
- Look for MongoDB connection errors or authentication failures

## Step 7: Keep Your App Updated

Whenever you make changes to your code:

1. Commit and push to GitHub:

   ```bash
   git add .
   git commit -m "Updated my awesome app"
   git push
   ```

2. Render will automatically detect the changes and redeploy your app!

## Step 8: Show Off Your Creation

Congratulations! Your app is now live on the internet. Time to celebrate:

1. Share the URL with friends and family
2. Add it to your resume/portfolio
3. Post about it on social media (optional, but you've earned bragging rights)
4. Maybe treat yourself to that fancy coffee you've been eyeing

## What's Next?

Now that you've conquered deployment with MongoDB, here are some ideas for your next steps:

- Implement user authentication and authorization
- Add data validation and error handling for your database operations
- Set up database indexes for better performance
- Add more complex API endpoints (search, filtering, pagination)
- Set up automated testing (including database tests)
- Learn about CI/CD pipelines
- Explore MongoDB's advanced features (aggregation pipelines, transactions)
- Consider adding Redis for caching frequently accessed data
- Explore Render's other services (background workers, cron jobs)

## Final Words

You did it! Your FastAPI app is now living its best life on the internet, ready to serve requests from anywhere in the world. Take a moment to appreciate what you've accomplished – you've gone from local development to production deployment, which is no small feat.

Remember, every experienced developer has been exactly where you are right now. We've all stared at error messages, wondered why things weren't working, and felt the incredible satisfaction of seeing "Build completed successfully" for the first time.

Keep building, keep learning, and most importantly, keep having fun with code. The internet is now a little bit more awesome because of your contribution.

Happy coding, and welcome to the world of deployed applications! 🚀

---

_Made with ❤️ for Curtin students learning web development._
