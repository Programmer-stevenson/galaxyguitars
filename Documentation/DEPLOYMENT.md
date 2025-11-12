# Galaxy Guitars - Deployment & Hosting Guide

## 🌐 Production Deployment Options

### Option 1: Vercel (Recommended for Frontend)

**Why Vercel?**
- ✅ Optimized for React/Vite
- ✅ Automatic HTTPS
- ✅ CDN worldwide
- ✅ Free tier available
- ✅ Zero configuration

**Steps:**

1. **Install Vercel CLI**
   ```bash
   npm install -g vercel
   ```

2. **Login to Vercel**
   ```bash
   vercel login
   ```

3. **Deploy**
   ```bash
   cd /home/claude/galaxy-guitars
   vercel
   ```

4. **Production Deployment**
   ```bash
   vercel --prod
   ```

5. **Custom Domain** (Optional)
   - Go to Vercel Dashboard
   - Select your project
   - Settings → Domains
   - Add: `www.galaxyguitar.com`

**Automatic Deployments:**
Connect your GitHub repository for automatic deployments on push.

---

### Option 2: Netlify

**Why Netlify?**
- ✅ Easy drag-and-drop
- ✅ Form handling
- ✅ Serverless functions
- ✅ Free SSL
- ✅ Great for static sites

**Steps:**

1. **Build the Project**
   ```bash
   npm run build
   ```

2. **Deploy via Netlify CLI**
   ```bash
   npm install -g netlify-cli
   netlify login
   netlify deploy --prod --dir=dist
   ```

3. **Or Deploy via Web Interface**
   - Go to [app.netlify.com](https://app.netlify.com)
   - Drag and drop the `dist` folder
   - Done!

**Build Settings for GitHub Integration:**
- Build command: `npm run build`
- Publish directory: `dist`
- Node version: `18`

---

### Option 3: AWS S3 + CloudFront

**For Enterprise Deployment**

**Steps:**

1. **Build the Project**
   ```bash
   npm run build
   ```

2. **Create S3 Bucket**
   ```bash
   aws s3 mb s3://galaxyguitar-website
   ```

3. **Upload Files**
   ```bash
   aws s3 sync dist/ s3://galaxyguitar-website --delete
   ```

4. **Configure Static Website Hosting**
   ```bash
   aws s3 website s3://galaxyguitar-website \
     --index-document index.html \
     --error-document index.html
   ```

5. **Create CloudFront Distribution**
   - Origin: S3 bucket
   - Default root: index.html
   - SSL Certificate: ACM

6. **Update DNS**
   - Point domain to CloudFront distribution

**Costs:** ~$5-20/month depending on traffic

---

### Option 4: DigitalOcean App Platform

**Balanced Option**

1. **Connect GitHub Repository**
2. **Configure Build**
   - Build Command: `npm run build`
   - Output Directory: `dist`
3. **Deploy**

**Cost:** $5-12/month

---

### Option 5: Traditional VPS (Nginx)

**Full Control Option**

**Prerequisites:**
- Ubuntu 22.04 VPS
- Domain name pointed to VPS IP

**Steps:**

1. **Connect to VPS**
   ```bash
   ssh root@your-vps-ip
   ```

2. **Install Nginx**
   ```bash
   apt update
   apt install nginx -y
   ```

3. **Install Node.js**
   ```bash
   curl -fsSL https://deb.nodesource.com/setup_18.x | bash -
   apt install nodejs -y
   ```

4. **Clone and Build**
   ```bash
   cd /var/www
   git clone your-repo galaxy-guitars
   cd galaxy-guitars
   npm install
   npm run build
   ```

5. **Configure Nginx**
   ```bash
   nano /etc/nginx/sites-available/galaxyguitar
   ```

   Add:
   ```nginx
   server {
       listen 80;
       server_name galaxyguitar.com www.galaxyguitar.com;
       root /var/www/galaxy-guitars/dist;
       index index.html;

       location / {
           try_files $uri $uri/ /index.html;
       }

       # Gzip compression
       gzip on;
       gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

       # Cache static assets
       location ~* \.(jpg|jpeg|png|gif|ico|css|js|svg)$ {
           expires 1y;
           add_header Cache-Control "public, immutable";
       }
   }
   ```

6. **Enable Site**
   ```bash
   ln -s /etc/nginx/sites-available/galaxyguitar /etc/nginx/sites-enabled/
   nginx -t
   systemctl reload nginx
   ```

7. **SSL Certificate (Let's Encrypt)**
   ```bash
   apt install certbot python3-certbot-nginx -y
   certbot --nginx -d galaxyguitar.com -d www.galaxyguitar.com
   ```

8. **Auto-renewal Setup**
   ```bash
   certbot renew --dry-run
   ```

---

## 🔄 Continuous Deployment (CI/CD)

### GitHub Actions

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy to Production

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        
    - name: Install Dependencies
      run: npm ci
      
    - name: Build
      run: npm run build
      
    - name: Deploy to Vercel
      uses: amondnet/vercel-action@v20
      with:
        vercel-token: ${{ secrets.VERCEL_TOKEN }}
        vercel-org-id: ${{ secrets.ORG_ID }}
        vercel-project-id: ${{ secrets.PROJECT_ID }}
        vercel-args: '--prod'
```

---

## 🔍 Pre-Deployment Checklist

### Performance
- [ ] Run `npm run build` successfully
- [ ] Test production build locally: `npm run preview`
- [ ] Check Lighthouse scores (aim for 90+)
- [ ] Optimize images (compress, use WebP)
- [ ] Test on slow 3G connection

### SEO
- [ ] Verify meta tags
- [ ] Test Open Graph tags
- [ ] Add robots.txt
- [ ] Create sitemap.xml
- [ ] Test social media cards

### Security
- [ ] Enable HTTPS
- [ ] Set security headers
- [ ] Update dependencies
- [ ] Remove console.logs
- [ ] Validate environment variables

### Testing
- [ ] Test on Chrome, Firefox, Safari, Edge
- [ ] Test on mobile devices
- [ ] Verify all links work
- [ ] Test contact forms
- [ ] Check 404 pages

### Analytics
- [ ] Set up Google Analytics
- [ ] Configure error tracking (Sentry)
- [ ] Test tracking events

---

## 📊 Post-Deployment Monitoring

### Tools to Use

1. **Google Analytics**
   - Traffic monitoring
   - User behavior
   - Conversion tracking

2. **Google Search Console**
   - SEO performance
   - Index coverage
   - Search queries

3. **Uptime Monitoring**
   - UptimeRobot (free)
   - Pingdom
   - StatusCake

4. **Performance Monitoring**
   - Lighthouse CI
   - WebPageTest
   - GTmetrix

---

## 🚨 Troubleshooting Deployment Issues

### Build Fails

**Error: Out of memory**
```bash
# Increase Node.js memory limit
NODE_OPTIONS=--max_old_space_size=4096 npm run build
```

**Error: Module not found**
```bash
# Clear cache and reinstall
rm -rf node_modules package-lock.json
npm install
```

### Site Not Loading

**Check DNS:**
```bash
dig galaxyguitar.com
nslookup galaxyguitar.com
```

**Check SSL:**
```bash
openssl s_client -connect galaxyguitar.com:443
```

### Three.js Performance Issues

**Solutions:**
1. Reduce particle count in NebulaBackground
2. Lower texture resolutions
3. Enable hardware acceleration
4. Implement lazy loading

---

## 💾 Backup Strategy

### Automated Backups

**Daily Backups Script:**
```bash
#!/bin/bash
DATE=$(date +%Y%m%d)
tar -czf backup-$DATE.tar.gz /var/www/galaxy-guitars
aws s3 cp backup-$DATE.tar.gz s3://backup-bucket/
```

**Database Backups** (when using MongoDB):
```bash
mongodump --uri="mongodb://..." --out=/backups/
```

---

## 📈 Scaling Considerations

### When to Scale

- Site gets 10,000+ daily visitors
- Response times > 2 seconds
- High CPU/memory usage
- Database queries slow

### Scaling Options

1. **CDN** - CloudFlare, Fastly
2. **Load Balancer** - Multiple servers
3. **Database** - MongoDB Atlas, AWS RDS
4. **Caching** - Redis, Memcached

---

## 💰 Cost Estimates

### Hosting Costs

| Option | Monthly Cost | Traffic Limit |
|--------|--------------|---------------|
| Vercel | $0 - $20 | 100GB |
| Netlify | $0 - $19 | 100GB |
| DigitalOcean | $5 - $12 | 1TB |
| AWS S3+CloudFront | $5 - $50 | Variable |
| VPS (Nginx) | $5 - $20 | Variable |

### Domain Costs
- .com domain: $10-15/year
- Privacy protection: $5-10/year

---

## 🎯 Production Optimization Tips

1. **Enable Compression**
   - Gzip all text files
   - Use WebP for images

2. **Implement Caching**
   - Browser caching headers
   - Service worker for offline

3. **Code Splitting**
   - Already configured in Vite
   - Lazy load routes

4. **CDN Usage**
   - Serve static assets from CDN
   - Use CloudFlare for free CDN

5. **Database Optimization** (future)
   - Index frequently queried fields
   - Use connection pooling
   - Implement caching layer

---

## 🔗 Useful Resources

- [Vite Deployment Guide](https://vitejs.dev/guide/static-deploy.html)
- [React Production Checklist](https://react.dev/learn/production-checklist)
- [Web Performance Best Practices](https://web.dev/performance/)
- [SSL/TLS Configuration](https://ssl-config.mozilla.org/)

---

**Need Help?** Contact: info@galaxyguitar.com
