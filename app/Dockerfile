# Use a imagem oficial do Nginx como base
FROM nginx:latest

# Copie os arquivos HTML e CSS para o diretório padrão do Nginx

COPY index.html /usr/share/nginx/html/
COPY style.css /usr/share/nginx/html/
COPY foto.png /usr/share/nginx/html/
# Exponha a porta 80
EXPOSE 80

# O Nginx inicia automaticamente como o comando padrão da imagem base
