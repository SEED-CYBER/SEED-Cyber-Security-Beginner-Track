# Module 0: Introduction & Cloud Providers

## What is a Cloud VPS?

A Virtual Private Server (VPS) is a virtualized server that acts as your own isolated computer on the internet. Unlike shared hosting, a VPS gives you:

- **Root access:** Full control over your system
- **Dedicated resources:** Guaranteed CPU, RAM, and storage
- **Security:** Your own firewall and network isolation
- **Flexibility:** Install any software you need

## Why We Use Cloud VPS for This Training

Throughout this 6-week internship, you'll deploy real services on a VPS:
- Linux server hardening requires a real Linux system
- Security monitoring tools need live network traffic
- Offensive security exercises require isolated network environments
- Shared workspace isolation prevents students from interfering with each other

## Choosing Your Cloud Provider

### Popular VPS Providers

| Provider | Best For | Free Tier | $/month |
|----------|----------|-----------|---------|
| **DigitalOcean** | Beginners, clean interface | $5/month trial | $5+ |
| **AWS (EC2)** | Scalability, enterprise | 1-year free tier | $2-10 |
| **Linode** | Developers, great support | $5 credit | $5+ |
| **Vultr** | High performance, global | $2.50/month | $2.50+ |
| **Azure** | Windows/Enterprise | Free tier | $15+ |
| **Hetzner** | Budget-friendly, Europe | Minimal | $3+ |

### Recommendation for This Internship

**DigitalOcean** or **Linode** are recommended because:
- Affordable and straightforward pricing
- Excellent Linux documentation
- Easy-to-use control panels
- Good performance for security training
- Abundant community examples online

**Minimum specifications needed:**
- 1 vCPU
- 1-2 GB RAM
- 20-25 GB SSD storage
- Ubuntu 20.04 LTS or 22.04 LTS

## Account Setup Steps

### 1. Choose Your Provider
Compare the options above. Consider:
- Budget (most providers have free trials or credits)
- Geographic location (choose a region near you)
- Support availability
- Documentation quality

### 2. Create an Account

**For DigitalOcean:**
1. Go to https://www.digitalocean.com
2. Sign up with email (use your personal email)
3. Add payment method
4. You'll receive free credits ($5-10 depending on promotions)

**For Linode:**
1. Go to https://www.linode.com
2. Create account
3. Add payment method
4. Look for promotional codes for $5-10 credit

**For AWS:**
1. Go to https://aws.amazon.com
2. Create account with root email
3. Set up Free Tier eligibility
4. Note: Free tier has monthly limits and time restrictions

### 3. Verify Your Account

After account creation:
- Confirm your email address
- Set up two-factor authentication (recommended)
- Verify billing information
- Review your account limits and quotas

## Understanding VPS Regions

When deploying your VPS, you'll choose a region (data center location):

```
Regions = Phyiscal data center locations worldwide
```

**Why region matters:**
- **Latency:** Closer = faster response
- **Compliance:** Some data must stay in specific countries
- **Availability:** Some regions have more servers
- **Cost:** Varies slightly by region

**For this internship:** Choose a region geographically close to you or choose a major hub (us-east, eu-central, sg for Asia).

## Operating System Choice

You'll be deploying **Ubuntu Linux**, the industry standard for servers.

**Available versions:**
- **Ubuntu 20.04 LTS:** Stable, proven in production (good for this course)
- **Ubuntu 22.04 LTS:** Newest LTS, recommended if available
- **Debian:** Alternative, very similar

**LTS = Long Term Support** - receives updates for 5 years.

## Deployment Checklist

Before deploying, prepare:

- [ ] Cloud provider account created
- [ ] Billing method verified
- [ ] Region selected (geographically close preferred)
- [ ] Ubuntu 20.04 or 22.04 LTS chosen as image
- [ ] Instance type selected (minimum 1 vCPU, 1-2GB RAM)
- [ ] Storage size confirmed (20-25 GB minimum)

## Next Steps

Once your account is set up:
1. Write down your account details somewhere safe
2. Read [Part-1-Development-Tools.md](Part-1-Development-Tools.md)
3. Come back to actually deploy the VPS in the exercises

## Important Security Notes

🔒 **Never share your account credentials**  
🔒 **Keep your password strong and unique**  
🔒 **Store billing information securely**  
🔒 **Check your account regularly for unexpected charges**  
🔒 **Set up billing alerts if available**

---

**Next:** [Module 1: Development Tools & Environment Setup](Part-1-Development-Tools.md)
