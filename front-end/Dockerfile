# front-end/Dockerfile
FROM node:20-alpine AS build
 
WORKDIR /app
 
COPY package.json package-lock.json ./
RUN npm ci
 
COPY . .
RUN npm run build -- --configuration=development
 
# --- Stage 2: Serve with Nginx ---
FROM nginx:alpine
 
COPY nginx.conf /etc/nginx/conf.d/default.conf
 
# Using a wildcard match to grab the contents of the generated folder 
# regardless of whether it's named split-screen-app or yarls-ai
COPY --from=build /app/dist/* /usr/share/nginx/html
 
EXPOSE 80
 
CMD ["nginx", "-g", "daemon off;"]
