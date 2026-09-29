@'
# Menu Restaurante - Frontend

Frontend en Vue 3 que consume la API REST del backend (menu-restaurante) para gestionar las resenas de los platos.

Este proyecto es independiente del backend, se conecta a el por HTTP sin modificarlo.

## Que hace

- Muestra el listado de resenas guardadas en MongoDB
- Permite crear una resena nueva
- Permite editar una resena existente
- Permite eliminar una resena
- Todo se actualiza al instante sin recargar la pagina

## Como funciona

El frontend corre en localhost:5173 y el backend en localhost:3001.

Las peticiones que hace este proyecto a /api/resenas se redirigen automaticamente al backend gracias a un proxy configurado en vite.config.js, asi se evitan problemas de CORS sin tocar el backend.

## Como correrlo

1. Instalar dependencias
npm install


2. Asegurarse de que el backend (menu-restaurante) este corriendo en el puerto 3001

3. Iniciar el frontend
npm run dev


4. Abrir http://localhost:5173

## Backend relacionado

https://github.com/RicarDr21/menu-restaurante
'@ | Set-Content -Path README.md -Encoding utf8
