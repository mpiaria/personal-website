# AWS Amplify Hosting Plan for Next.js Site

## Context

The user wants to host their Next.js site located in `./packages/ui` using AWS Amplify, with CDK infrastructure code in `./infrastructure`.

### Current State

| Component | Status |
|-----------|--------|
| `packages/ui` | Next.js 16.3.4 (App Router), build output in `.next/` |
| `infrastructure/` | Empty CDK v2 skeleton, no resources defined |
| `packages/ui/` | No Amplify-specific configuration |
| CI/CD | Basic lint/test workflow, no deployment |

### What is AWS Amplify?

AWS Amplify is a complete solution that provides:
- **Hosting**: Managed hosting with global CDN, automatic scaling
- **CI/CD**: Built-in continuous deployment from GitHub
- **Backend**: Managed backend services (optional, not required for static hosting)

For a Next.js site, you have two main options:
1. **Amplify Hosting only** - Use Amplify for hosting (not backend)
2. **Full Amplify** - Use both hosting and backend services

---

## Approach 1: Amplify Hosting (Recommended for Static Next.js)

This approach uses Amplify Hosting to serve your pre-built Next.js static assets.

### What it does:
- Builds your Next.js app (`next build` + `next export` or `next start` as server)
- Hosts on AWS CloudFront with HTTPS
- Automatic HTTPS with ACM certificates
- Continuous deployment from GitHub

### CDK Implementation

```typescript
// infrastructure/src/lib/amplify-stack.ts
import * as cdk from "aws-cdk-lib";
import { Construct } from "constructs";
import * as amplify from "aws-cdk-lib/aws-amplify";

export class AmplifyStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // Build the UI app first (run this locally or in CI before deploying)
    // npm run build --workspace=packages/ui
    
    const app = new amplify.App(this, "PersonalWebsiteApp", {
      sourceCodeProvider: new amplify.GitHubSourceCodeProvider({
        owner: "your-github-username",
        repository: "personal-website",
        oAuthConfiguration: {
          scopes: ["repo", "admin:repo_hook"],
        },
      }),
      buildSpec: amplify.BuildSpec.fromObjectToYaml({
        version: "1",
        frontend: {
          phases: {
            preBuild: {
              commands: [
                "npm ci",
                "npm run build --workspace=packages/ui", // or cd packages/ui && npm ci && npm run build
              ],
            },
            build: {
              commands: [
                "npx next export --out-dir out", // Export static site
              ],
            },
          },
          artifacts: {
            baseDirectory: "out",
            files: ["**/*"],
          },
          cache: {
            paths: ["node_modules/**/*"],
          },
        },
      }),
      environmentVariables: {
        // Add any env vars your Next.js app needs
      },
      customHeaders: [
        { header: "Strict-Transport-Security", value: "max-age=31536000; includeSubDomains; preload" },
        { header: "X-Frame-Options", value: "DENY" },
        { header: "X-Content-Type-Options", value: "nosniff" },
      ],
    });

    // Create a custom domain mapping (if you have one)
    // const domain = app.addDomain("your-domain.com", {
    //   enableAutoSubDomain: true,
    // });
  }
}
```

### Prerequisites:
1. GitHub repository with your code
2. GitHub OAuth token with repo access
3. Next.js configured for static export (`next export`) or server-side deployment

### Commands to Deploy:

```bash
# 1. Add Amplify CDK package
npm install --save-dev aws-cdk-lib@2.268.0

# 2. Create the stack file (see above)

# 3. Deploy
cd infrastructure
npx cdk deploy
```

---

## Approach 2: Amplify with Full Backend

This approach uses Amplify's full stack capabilities including managed backend services.

### What it adds:
- API Gateway + Lambda for serverless backend
- Cognito for authentication
- S3 for file storage
- DynamoDB for database
- Push notifications, analytics, etc.

### When to use:
- You need a backend (API, auth, database)
- You want Amplify's CLI for local development
- You want managed backend services

---

## Approach 3: Traditional S3 + CloudFront (Alternative)

Instead of Amplify, use S3 + CloudFront directly:

| Service | Purpose |
|---------|---------|
| S3 | Static site hosting |
| CloudFront | CDN with SSL |
| ACM | SSL certificate |
| Route 53 | DNS (optional) |

This is simpler if you don't need Amplify's CI/CD or app management features.

---

## Key Considerations

### 1. Next.js Deployment Mode

**Static Export (`next export`)**:
- Works with Amplify Hosting
- No server-side rendering at runtime
- All routes must be known at build time

**Server-Side Deployment**:
- Use `next start` as the start command
- Requires more resources but supports SSR

### 2. Build Process

Amplify will run your build commands, but you need to ensure:
```yaml
# buildSpec in CDK
version: 1
frontend:
  phases:
    preBuild:
      commands: ["npm ci"]
    build:
      commands: ["npx next build"]
  artifacts:
    baseDirectory: ".next"
    files:
      - "**/*"
  cache:
    paths:
      - "node_modules/**/*"
```

### 3. Environment Variables

Pass to Amplify via `environmentVariables` in CDK or through the Amplify Console.

---

## Recommended Steps

1. **Start with Approach 1** (Amplify Hosting only)
2. **Configure your Next.js app** for the deployment mode you want
3. **Write the CDK stack** in `infrastructure/src/lib/amplify-stack.ts`
4. **Test deployment** with `npx cdk deploy`
5. **Configure GitHub OAuth** in Amplify Console for CI/CD

---

## Files to Create/Modify

| Action | File |
|--------|------|
| Create | `infrastructure/src/lib/amplify-stack.ts` |
| Modify | `infrastructure/src/bin/infrastructure.ts` (add AmplifyStack) |
| Create (optional) | `infrastructure/src/lib/env.ts` for config |

---

## Cost Considerations

- **Amplify Hosting**: Free tier available, then ~$10-20/month depending on traffic
- **CloudFront**: ~$0.085 per GB (first 10 TB)
- **ACM**: Free (for certificates)
