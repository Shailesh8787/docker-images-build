# Pull node js image latest version default
FROM node as build-stage

# Get http_proxy form arguments
ARG  HTTP_PROXY

# Set Http and https proxy
RUN export http_proxy=${HTTP_PROXY}\
&& export https_proxy=${HTTP_PROXY}

# Echo http_proxy
RUN echo $http_proxy

# Remove lock file
RUN rm -rf /var/lib/apt/lists/lock

# Check npm proxy list, If not exists npm proxy then save proxy
RUN npm config list \
&& npm config set proxy=${HTTP_PROXY} \
&& npm config set https-proxy=${HTTP_PROXY}
RUN npm config list

# Vue Cli Installation
RUN npm install -g @vue/cli

# Creating directories
#/var/www/html/docker-vue
WORKDIR /var/www/html/docker-vue

# Copy Package.json file
COPY package*.json ./

# Installing all dependencies
RUN npm install

# Copy all file and folder current directories
COPY . .

# Build Application
RUN npm run build

FROM nginx:1.15.8-alpine as production-build

COPY --from=build-stage /var/www/html/docker-vue/dist  /usr/share/nginx/html

# Expose prot your application run witch port
EXPOSE 8003

# Serving file
CMD ["nginx", "-g", "daemon off;"]


# Build command docker build --build-arg=your-proxy -t your-tagname
# Run Docker file docker run --ti -p witchport:your localport --rm --name your docker-container new container
