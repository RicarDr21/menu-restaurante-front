# Menu Restaurante - Frontend

Frontend en Vue 3 (Vite) que consume la API REST del backend (menu-restaurante) para mostrar el menu y gestionar las resenas de los platos.

Este proyecto es independiente del backend, se conecta a el por HTTP sin modificarlo.

## Que hace

- Muestra el menu de platos del restaurante (solo lectura, consume GET /api/platos)
- Muestra el listado de resenas guardadas en MongoDB
- Permite crear una resena nueva
- Permite editar una resena existente
- Permite eliminar una resena
- Todo se actualiza al instante sin recargar la pagina

## Como funciona

El frontend corre en localhost:5173 y el backend en localhost:3001.

El backend tiene CORS habilitado (paquete cors) autorizando al origen localhost:5173, asi que el frontend le habla directamente por su URL completa (http://localhost:3001/api/resenas y http://localhost:3001/api/platos), sin depender de un proxy.

## Como correrlo

1. Instalar dependencias

npm install

2. Asegurarse de que el backend (menu-restaurante) este corriendo en el puerto 3001

3. Iniciar el frontend

npm run dev

4. Abrir http://localhost:5173

## Backend relacionado
https://github.com/RicarDr21/menu-restaurante
