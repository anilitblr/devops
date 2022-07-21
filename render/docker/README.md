# Deploy Python FastAPI app

- [Navigate to Render Dashboard](https://dashboard.render.com/)
- Choose **New +** and choose **Web Service**
- Connect a repository - Search for a repository from which you would like to deploy and click on **Connect**
- Name: **example-app-database**
- Environment:
- Region: **Oregon (US West)**
- Branch: Choose **main** or **master**
- Plan: **Starter**
- Click on **Advanced**
- Add Environment Variable: **key : value**  
  - ex: **DATABASE_USER : test**
- Health Check Path: **/**
- Docker Build Context Directory: **.**
- Dockerfile Path: **.**
- Docker Command: 
- Auto-Deploy: **Yes**
- Click on **Create Web Service**

# Reference

https://render.com/docs/docker
