# How to update next.js project source code on Ubuntu server manually
## Steps to update your Next.js project:

1. **Connect to your server** - Use SSH to access your Ubuntu server
2. **Navigate to project directory** - Go to the location where your Next.js application is deployed
3. **Pull latest code** - Fetch and merge the latest changes from your Git repository
4. **Install dependencies** - Update any new or modified npm packages
5. **Build the project** - Compile your Next.js application for production
6. **Check PM2 status** - Verify your running processes
7. **Restart the application** - Apply the changes by restarting your PM2 process

## Example commands:
```bash
# Login to your server
ssh user@your-server-ip

# Navigate to your project directory
cd /var/www/pphat/pphat.stackdev.com

# Pull the latest changes from Git base on your branch
git pull

# Install new dependencies
npm install

# Rebuild the project
npm run build

# Check PM2 process list
pm2 status

# Restart the next.js application using PM2 (replace 'pphat.stackdev.com' with your actual PM2 process name)
pm2 restart pphat.stackdev.com
```
