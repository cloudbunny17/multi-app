# Get a base Node image
FROM node:16-alpine

# Make the 'app' folder the current working directory
WORKDIR /app

# Copy both 'package.json' and 'package-lock.json' (if present)
COPY app/vue/package.json ./
COPY app/vue/package-lock.json ./

# Install project dependencies
RUN npm install

# Copy project files and folders to the current working directory (i.e. 'app' folder)
COPY ./app/vue .

# Build app for production with minification
RUN npm run build

# Runs the website on http://localhost:8080
CMD ["npm", "run", "serve"]
