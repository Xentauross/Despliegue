# 1. Usamos una versión de Node estable y ligera
FROM node:18-alpine

# 2. Definimos la carpeta de trabajo
WORKDIR /app

# 3. Copiamos los archivos de configuración
COPY package.json .
# (Omitimos el package-lock.json estricto para evitar errores de compatibilidad Windows/Linux)

# 4. Instalamos las dependencias de forma flexible
RUN npm install

# 5. Copiamos el resto del código de la web
COPY . .

# 6. Exponemos el puerto
EXPOSE 3000

# 7. Arrancamos la aplicación
CMD ["npm", "start"]