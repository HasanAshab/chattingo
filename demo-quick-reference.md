# Chattingo Demo - Quick Reference Card

## 🚀 Pre-Demo Checklist
- [ ] Docker and Docker Compose running
- [ ] Jenkins accessible and pipelines configured
- [ ] VPS accessible via SSH
- [ ] Multiple browser windows/tabs ready
- [ ] Test user accounts prepared
- [ ] Screen recording software ready

## ⚡ Quick Commands

### Local Setup
```bash
# Start application
docker-compose up --build -d

# Check status
docker-compose ps

# View logs
docker-compose logs -f

# Stop application
docker-compose down
```

### Jenkins Pipeline
```bash
# Trigger pipeline (make a change and push)
git add . && git commit -m "Demo change" && git push

# Check pipeline status
# Navigate to Jenkins UI -> chattingo folder
```

### Production Deployment
```bash
# SSH to VPS
ssh -i ~/.ssh/chattingo_vps root@72.60.111.27

# Check production containers
docker ps

# View production logs
docker-compose logs -f
```

## 🎯 Key Demo Points

### Part 1: Docker Setup (1.5 min)
- Show project structure
- Explain docker-compose.yml
- Start services with health checks
- Show container networking

### Part 2: Jenkins Pipeline (2 min)
- Orchestrator pattern explanation
- Change detection demo
- Parallel execution showcase
- Security scanning highlights

### Part 3: Application Walkthrough (2 min)
- User registration/login
- JWT authentication
- Dashboard overview
- Basic navigation

### Part 4: Key Features (2.5 min)
- Real-time messaging (WebSocket)
- User management
- Group chat functionality
- Responsive design
- Security features

### Part 5: Production Deployment (1.5 min)
- VPS environment
- Automated deployment
- Live production app
- Infrastructure overview

## 🔧 Troubleshooting

### If Docker fails:
```bash
docker system prune -f
docker-compose down -v
docker-compose up --build -d
```

### If Jenkins pipeline fails:
- Check webhook configuration
- Verify credentials
- Check Jenkins logs

### If application doesn't load:
- Check port availability (3000, 8080)
- Verify environment variables
- Check database connection

## 📱 URLs to Remember
- Local Frontend: http://localhost:3000
- Local Backend: http://localhost:8080
- Jenkins: http://your-jenkins-url
- Production: http://your-production-url

## 🎬 Recording Settings
- Resolution: 1080p (1920x1080)
- Frame Rate: 30fps
- Audio: System + Microphone
- Duration: 8-10 minutes total

## 💡 Pro Tips
- Practice the demo 2-3 times
- Have backup screenshots ready
- Keep energy level high
- Explain before you click
- Highlight unique features
- End with strong summary
