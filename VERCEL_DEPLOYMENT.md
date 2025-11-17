# Deploying RSSHub to Vercel

This guide will help you deploy RSSHub to your own Vercel infrastructure.

## Prerequisites

- A [Vercel account](https://vercel.com/signup)
- [Vercel CLI](https://vercel.com/cli) installed (optional, for local deployment)
- Node.js 22.x or higher
- pnpm package manager

## Quick Start

### Option 1: Deploy via Vercel Dashboard (Recommended)

1. **Fork or clone this repository** to your GitHub account

2. **Import to Vercel:**
   - Go to [Vercel Dashboard](https://vercel.com/dashboard)
   - Click "Add New..." → "Project"
   - Import your RSSHub repository
   - Configure the project:
     - **Framework Preset:** Other
     - **Build Command:** `pnpm build:vercel`
     - **Output Directory:** `dist`
     - **Install Command:** `pnpm install`

3. **Configure Environment Variables:**
   - Go to "Settings" → "Environment Variables"
   - Add required variables (see [Environment Variables](#environment-variables) section)
   - Minimum required:
     ```
     NODE_ENV=production
     CACHE_TYPE=memory
     PUPPETEER_SKIP_DOWNLOAD=1
     NO_LOGFILES=true
     ```

4. **Deploy:**
   - Click "Deploy"
   - Wait for the build to complete
   - Your RSSHub instance will be available at `https://your-project.vercel.app`

### Option 2: Deploy via Vercel CLI

1. **Install Vercel CLI:**
   ```bash
   npm i -g vercel
   ```

2. **Login to Vercel:**
   ```bash
   vercel login
   ```

3. **Deploy from project directory:**
   ```bash
   cd /path/to/RSSHub
   vercel
   ```

4. **Follow the prompts:**
   - Set up and deploy: Yes
   - Which scope: Select your account/team
   - Link to existing project: No (or Yes if you already created one)
   - Project name: your-rsshub (or custom name)
   - Directory: ./
   - Override settings: No (vercel.json is already configured)

5. **Add environment variables:**
   ```bash
   vercel env add NODE_ENV production
   vercel env add CACHE_TYPE memory
   vercel env add PUPPETEER_SKIP_DOWNLOAD 1
   vercel env add NO_LOGFILES true
   ```

6. **Deploy to production:**
   ```bash
   vercel --prod
   ```

## Configuration

### vercel.json

The project includes a pre-configured `vercel.json` file with the following settings:

- **Build Command:** `pnpm build:vercel`
- **Output Directory:** `dist`
- **Runtime:** Node.js 22.x
- **Memory:** 1024 MB
- **Max Duration:** 60 seconds
- **Routes:** All requests are routed to the serverless function

### Environment Variables

Copy `.env.vercel.example` as a reference for available environment variables.

#### Required Variables

| Variable | Value | Description |
|----------|-------|-------------|
| `NODE_ENV` | `production` | Node environment |
| `CACHE_TYPE` | `memory` or `redis` | Cache backend type |
| `PUPPETEER_SKIP_DOWNLOAD` | `1` | Skip Chromium download |
| `NO_LOGFILES` | `true` | Disable file logging |

#### Recommended Variables

| Variable | Example | Description |
|----------|---------|-------------|
| `ACCESS_KEY` | `your-secret-key` | API access key for security |
| `LOGGER_LEVEL` | `info` | Logging level |
| `REQUEST_TIMEOUT` | `30000` | Request timeout in ms |
| `REDIS_URL` | `redis://host:port` | Redis connection (if using Redis cache) |

#### Optional Route-Specific Variables

Add only the API keys/credentials for routes you need:

- `GITHUB_ACCESS_TOKEN` - For GitHub routes
- `TWITTER_USERNAME`, `TWITTER_PASSWORD` - For Twitter/X routes
- `YOUTUBE_KEY` - For YouTube routes
- `PIXIV_REFRESHTOKEN` - For Pixiv routes
- See `.env.vercel.example` for complete list

### Caching Strategy

#### Memory Cache (Free Tier)

Good for testing and low-traffic deployments:

```
CACHE_TYPE=memory
MEMORY_MAX=256
```

**Limitations:**
- Cache is lost on function cold starts
- Limited memory availability
- Not shared across function instances

#### Redis Cache (Recommended for Production)

For production deployments, use Redis for persistent caching:

```
CACHE_TYPE=redis
REDIS_URL=redis://username:password@host:port/
```

**Options:**
- [Vercel KV](https://vercel.com/docs/storage/vercel-kv) (integrated Redis)
- [Upstash Redis](https://upstash.com/) (serverless Redis)
- Self-hosted Redis instance

## Vercel Limits & Optimization

### Hobby (Free) Plan Limits

- **Function Execution:** 10 seconds max
- **Function Size:** 50 MB max
- **Monthly Bandwidth:** 100 GB
- **Monthly Function Invocations:** Unlimited

### Pro Plan Limits

- **Function Execution:** 60 seconds max (configurable in `vercel.json`)
- **Function Size:** 250 MB max
- **Monthly Bandwidth:** 1 TB
- **Monthly Function Invocations:** Unlimited

### Optimization Tips

1. **Use Redis Caching:**
   - Reduces function execution time
   - Shares cache across instances
   - Set appropriate cache expiration times

2. **Optimize Feed Routes:**
   - Some routes may require longer execution times
   - Consider using external browser services for Puppeteer routes
   - Set `PUPPETEER_WS_ENDPOINT` to a service like [Browserless](https://browserless.io)

3. **Monitor Usage:**
   - Check Vercel Analytics for performance insights
   - Monitor function execution times
   - Track bandwidth usage

4. **Reduce Bundle Size:**
   - The build process already optimizes the bundle
   - Unnecessary dependencies are excluded via `build-vercel-packagejson.ts`

## Custom Domain

1. Go to your project in Vercel Dashboard
2. Navigate to "Settings" → "Domains"
3. Add your custom domain
4. Configure DNS according to Vercel's instructions
5. Wait for DNS propagation (can take up to 48 hours)

## Testing Your Deployment

After deployment, test your RSSHub instance:

```bash
# Test basic endpoint
curl https://your-project.vercel.app/

# Test a simple route (e.g., GitHub)
curl https://your-project.vercel.app/github/trending/daily

# Test with access key (if configured)
curl https://your-project.vercel.app/github/trending/daily?key=your-access-key
```

## Continuous Deployment

Vercel automatically deploys when you push to your repository:

- **Production Deployments:** Pushes to `main` branch
- **Preview Deployments:** Pull requests and other branches
- **Instant Rollback:** Revert to any previous deployment via dashboard

### GitHub Integration

Vercel provides automatic GitHub integration:

- Preview URLs for pull requests
- Deployment status checks
- Comment on PRs with deployment URL

## Troubleshooting

### Build Failures

**Issue:** Build fails with "Command failed: pnpm build:vercel"

**Solutions:**
- Check Node.js version is 22.x or higher in `vercel.json`
- Verify `pnpm-lock.yaml` is committed to repository
- Check build logs for specific errors

### Function Timeout

**Issue:** "Function execution timeout" errors

**Solutions:**
- Upgrade to Vercel Pro for 60-second timeout
- Use Redis caching to speed up responses
- Use external browser service for Puppeteer routes
- Optimize slow routes or disable them

### Cache Not Working

**Issue:** Slow response times, cache seems ineffective

**Solutions:**
- Verify `CACHE_TYPE` is set correctly
- Check `REDIS_URL` if using Redis
- Verify Redis connection and authentication
- Check cache expiration settings

### Memory Errors

**Issue:** "Function out of memory" errors

**Solutions:**
- Increase memory allocation in `vercel.json` (requires Pro plan)
- Reduce `MEMORY_MAX` for in-memory cache
- Switch to Redis caching
- Optimize heavy routes

### Routes Not Working

**Issue:** Specific routes return errors or 404

**Solutions:**
- Check if route requires API credentials (see `.env.vercel.example`)
- Verify environment variables are set correctly
- Check Vercel function logs for specific errors
- Some routes may require Puppeteer (needs external browser service)

## Security Best Practices

1. **Set an ACCESS_KEY:**
   ```
   ACCESS_KEY=your-long-random-secret-key
   ```
   Then access routes with: `?key=your-long-random-secret-key`

2. **Use Environment Variables:**
   - Never commit secrets to repository
   - Use Vercel's encrypted environment variables
   - Rotate keys regularly

3. **Configure CORS:**
   ```
   ALLOW_ORIGIN=https://yourdomain.com
   ```

4. **Monitor Usage:**
   - Enable Vercel Analytics
   - Set up error tracking with Sentry:
     ```
     SENTRY=https://your-sentry-dsn@sentry.io/project-id
     ```

5. **Rate Limiting:**
   - RSSHub includes built-in rate limiting
   - Consider using Vercel's Edge Config for additional protection

## Monitoring & Logging

### Vercel Logs

View real-time logs in Vercel Dashboard:
- Go to your project → "Logs"
- Filter by deployment, function, or time range
- Search for specific errors or patterns

### Sentry Integration

For advanced error tracking:

```bash
vercel env add SENTRY production
# Enter your Sentry DSN
```

### Custom Monitoring

RSSHub supports OpenTelemetry metrics:

```
OTEL_SECONDS_BUCKET=0.01,0.1,1,2,5,15,30,60
OTEL_MILLISECONDS_BUCKET=10,20,50,100,250,500,1000,5000,15000
```

## Advanced Configuration

### Using External Browser Service

For routes requiring Puppeteer (browser automation):

```
PUPPETEER_WS_ENDPOINT=wss://chrome.browserless.io?token=YOUR_TOKEN
```

Recommended services:
- [Browserless](https://browserless.io)
- [BrightData](https://brightdata.com)
- [Apify](https://apify.com)

### Multiple Proxy Support

For better reliability and avoiding rate limits:

```
PROXY_URIS=http://proxy1.example.com:8080,http://proxy2.example.com:8080
PROXY_STRATEGY=on_retry
```

### Custom Hotlink Template

For image proxying:

```
HOTLINK_TEMPLATE=https://images.weserv.nl/?url={{url}}
ALLOW_USER_HOTLINK_TEMPLATE=true
```

## Migration from Other Platforms

### From Docker

1. Export environment variables from Docker
2. Add them to Vercel environment variables
3. Deploy following the Quick Start guide
4. Update any URLs pointing to the old instance

### From Heroku

1. Export config vars: `heroku config -s > .env`
2. Review and adapt variables for Vercel
3. Import to Vercel: `vercel env add < .env`
4. Deploy following the Quick Start guide

## Cost Estimation

### Hobby (Free) Plan

- **Cost:** $0/month
- **Best for:** Personal use, testing, low traffic
- **Limitations:** 10-second timeout, no team features

### Pro Plan

- **Cost:** $20/month per user
- **Best for:** Production use, moderate to high traffic
- **Benefits:** 60-second timeout, analytics, teams, priority support

### Enterprise

- **Cost:** Custom pricing
- **Best for:** Large-scale deployments, SLA requirements
- **Contact:** Vercel sales team

## Support & Resources

- [RSSHub Documentation](https://docs.rsshub.app/)
- [RSSHub GitHub](https://github.com/DIYgod/RSSHub)
- [Vercel Documentation](https://vercel.com/docs)
- [Vercel Support](https://vercel.com/support)

## License

RSSHub is licensed under MIT License. See [LICENSE](LICENSE) for details.
