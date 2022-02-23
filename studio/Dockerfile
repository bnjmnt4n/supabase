## USAGE:
# Build:        docker build --target production -t supabase/studio:latest .
# Run:          docker run -p 3000:3000 supabase/studio
# Deploy:       docker push supabase/studio:latest
# Clean build:
#    docker build --target production --no-cache -t supabase/studio:latest .
#    docker builder prune


# Do `npm ci` separately so we can cache `node_modules`
# https://nodejs.org/en/docs/guides/nodejs-docker-webapp/
FROM node:14-alpine AS deps
RUN apk add --no-cache libc6-compat
WORKDIR /app
COPY package.json package-lock.json ./ 
RUN npm clean-install && npm cache clean --force

FROM node:14-alpine as builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
# NODE_OPTIONS because of OOM on low specs docker-host (1G RAM, 4G Swap) -> https://github.com/vercel/next.js/issues/32314#issuecomment-1046274967
RUN NEXT_STANDALONE=true \
    NODE_OPTIONS=--max_old_space_size=8192 \
    npm run build

# This leverages Next.js output file tracing to reduce image size:
# https://nextjs.org/docs/advanced-features/output-file-tracing
FROM node:14-alpine as production
WORKDIR /app
ENV NODE_ENV production
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs
COPY --from=builder /app/next.config.js ./
COPY --from=builder /app/public ./public
COPY --from=builder /app/package.json ./package.json
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000

CMD ["node", "server.js"]
