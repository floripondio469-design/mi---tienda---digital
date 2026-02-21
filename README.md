# mi---<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Tienda con Carrito</title>
    <style>
        :root { --primary: #2c3e50; --success: #25d366; --accent: #e67e22; }
        body { font-family: 'Segoe UI', sans-serif; background: #f4f7f6; margin: 0; padding: 20px; padding-bottom: 80px; }
        
        header { text-align: center; padding: 20px; background: var(--primary); color: white; border-radius: 10px; }
        
        /* Controles y Filtros */
        .controles { max-width: 800px; margin: 20px auto; text-align: center; }
        #buscador { width: 90%; padding: 12px; border-radius: 25px; border: 1px solid #ddd; margin-bottom: 10px; }
        .btn-cat { padding: 8px 15px; margin: 3px; border: none; border-radius: 15px; cursor: pointer; background: #ddd; }
        .btn-cat.activo { background: var(--accent); color: white; }

        /* Productos */
        .contenedor-productos { display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 20px; }
        .tarjeta { background: white; padding: 15px; border-radius: 12px; box-shadow: 0 4px 8px rgba(0,0,0,0.1); text-align: center; }
        .tarjeta img { width: 100%; height: 150px; object-fit: cover; border-radius: 8px; }
        
        /* Botón de Carrito Flotante */
        .carrito-flotante {
            position: fixed; bottom: 20px; right: 20px;
            background: var(--success); color: white; padding: 15px 25px;
            border-radius: 50px; font-weight: bold; cursor: pointer;
            box-shadow: 0 4px 15px rgba(0,0,0,0.3); z-index: 100;
        }

        /* Modal del Carrito */
        #modal-carrito {
            display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.7); z-index: 101;
        }
        .contenido-carrito {
            background: white; width: 90%; max-width: 500px; margin: 50px auto;
            padding: 20px; border-radius: 10px; max-height: 80vh; overflow-y: auto;
        }
        .item-carrito { display: flex; justify-content: space-between; border-bottom: 1px solid #eee; padding: 10px 0; }
        
        .btn-accion { padding: 8px; border: none; border-radius: 5px; cursor: pointer; width: 100%; margin-top: 10px; }
        .btn-add { background: var(--primary); color: white; }
        .btn-finalizar { background: var(--success); color: white; font-size: 1.1em; }
    </style>
</head>
<body>

<header>
    <h1>Mi Negocio Digital</h1>
</header>

<div class="controles">
    <input type="text" id="buscador" placeholder="Buscar..." onkeyup="filtrarTodo()">
    <div id="btns-categorias"></div>
</div>

<div class="contenedor-productos" id="catalogo"></div>

<div class="carrito-flotante" onclick="toggleCarrito()">
    🛒 Ver Pedido (<span id="cuenta-carrito">0</span>)
</div>

<div id="modal-carrito">
    <div class="contenido-carrito">
        <h2>Tu Pedido</h2>
        <div id="lista-carrito"></div>
        <h3>Total: $<span id="total-carrito">0</span></h3>
        <button class="btn-finalizar" onclick="enviarWhatsApp()">Enviar pedido por WhatsApp</button>
        <button class="btn-accion" style="background:#e74c3c; color:white;" onclick="toggleCarrito()">Cerrar</button>
    </div>
</div>

<script>
    const productos = [
        { id: 1, nombre: "Camiseta", precio: 25, cat: "Ropa", img: "https://images.unsplash.com/photo-1521572267360-ee0c2909d518?w=200" },
        { id: 2, nombre: "Reloj", precio: 90, cat: "Accesorios", img: "https://images.unsplash.com/photo-1524592091214-8c97af7c4a47?w=200" },
        { id: 3, nombre: "Audífonos", precio: 55, cat: "Gadgets", img: "https://images.unsplash.com/photo-1505740420928-5e560c06d30e?w=200" }
    ];

    let carrito = [];
    let categoriaActual = 'todos';

    function renderProductos() {
        const texto = document.getElementById('buscador').value.toLowerCase();
        const catalogo = document.getElementById('catalogo');
        catalogo.innerHTML = "";

        const filtrados = productos.filter(p => 
            p.nombre.toLowerCase().includes(texto) && (categoriaActual === 'todos' || p.cat === categoriaActual)
        );

        filtrados.forEach(p => {
            const div = document.createElement('div');
            div.className = 'tarjeta';
            div.innerHTML = `
                <img src="${p.img}">
                <h3>${p.nombre}</h3>
                <p>$${p.precio}</p>
                <button class="btn-accion btn-add" onclick="agregarAlCarrito(${p.id})">Añadir al carrito</button>
            `;
            catalogo.appendChild(div);
        });
    }

    function agregarAlCarrito(id) {
        const producto = productos.find(p => p.id === id);
        const existe = carrito.find(item => item.id === id);

        if (existe) {
            existe.cantidad++;
        } else {
            carrito.push({ ...producto, cantidad: 1 });
        }
        actualizarUI();
    }

    function actualizarUI() {
        document.getElementById('cuenta-carrito').innerText = carrito.reduce((sum, item) => sum + item.cantidad, 0);
        const lista = document.getElementById('lista-carrito');
        lista.innerHTML = "";
        let total = 0;

        carrito.forEach(item => {
            total += item.precio * item.cantidad;
            lista.innerHTML += `
                <div class="item-carrito">
                    <span>${item.cantidad}x ${item.nombre}</span>
                    <span>$${item.precio * item.cantidad}</span>
                </div>
            `;
        });
        document.getElementById('total-carrito').innerText = total;
    }

    function toggleCarrito() {
        const modal = document.getElementById('modal-carrito');
        modal.style.display = modal.style.display === 'block' ? 'none' : 'block';
    }

    function enviarWhatsApp() {
        const numero = "521234567890"; // CAMBIA ESTO POR TU NÚMERO
        let mensaje = "Hola! Quisiera hacer el siguiente pedido:\n\n";
        carrito.forEach(item => {
            mensaje += `- ${item.cantidad}x ${item.nombre} ($${item.precio * item.cantidad})\n`;
        });
        mensaje += `\n*Total a pagar: $${document.getElementById('total-carrito').innerText}*`;
        
        const url = `https://wa.me/${numero}?text=${encodeURIComponent(mensaje)}`;
        window.open(url, '_blank');
    }

    function filtrarTodo() { renderProductos(); }
    renderProductos();
</script>
</body>
</html>
tienda---digital
