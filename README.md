# Nobel-Uz1
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>Nobel_UZ</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; font-family: Arial, sans-serif; }
        body { background-color: #f8f8f8; color: #333; }
        header { background-color: #111; color: white; padding: 15px 20px; display: flex; justify-content: space-between; align-items: center; }
        .logo { display: flex; align-items: center; }
        .logo img { width: 50px; height: 50px; margin-right: 10px; }
        nav a { color: white; margin-left: 20px; text-decoration: none; font-weight: bold; transition: color 0.3s; }
        nav a:hover { color: #ff9800; }
        main { padding: 20px; }
        h2 { text-align: center; margin-bottom: 20px; }
        .products { display: grid; grid-template-columns: repeat(auto-fill, minmax(220px, 1fr)); gap: 20px; }
        .product { background-color: white; border-radius: 10px; padding: 15px; text-align: center; box-shadow: 0 4px 8px rgba(0,0,0,0.1); transition: transform 0.3s; }
        .product:hover { transform: scale(1.05); }
        .product img { width: 100%; border-radius: 10px; }
        .small { font-size: 13px; color: #666; }
        .btn { margin-top: 10px; padding: 8px 15px; border: none; background-color: #ff9800; color: white; font-size: 14px; border-radius: 5px; cursor: pointer; transition: background 0.3s; }
        .btn:hover { background-color: #e68900; }
        footer { background-color: #111; color: white; text-align: center; padding: 15px; margin-top: 20px; }
        #cart-modal { display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.5); justify-content: center; align-items: center; z-index: 1000; }
        #cart-modal.show { display: flex; }
        .cart-item { display:flex; gap:10px; align-items:center; }
        .cart-item img{width:48px;height:48px;border-radius:6px;object-fit:cover}
    </style>
</head>
<body>

<header>
    <div class="logo">
        <img src="images/logo.png" alt="Nobel_UZ Logo" />
        <h1>Nobel_UZ</h1>
    </div>
    <nav>
        <a href="https://www.instagram.com/nobel_uz1" target="_blank" rel="noopener noreferrer">Instagram</a>
        <a href="https://t.me/+X5m5jCOv7tVlMmYy" target="_blank" rel="noopener noreferrer">Telegram</a>
        <a href="tel:+998950670856">Позвонить</a>
        <div style="color:white; margin-left:20px; display: inline-block;">
            Корзина: <span id="cart-count">0</span>
        </div>
        <button onclick="showCart()" class="btn" style="margin-left: 10px; padding: 5px 10px;">Показать корзину</button>
    </nav>
</header>

<main>
    <h2>Наши товары</h2>
    <div class="products" id="products"></div>
</main>

<!-- Модальное окно корзины -->
<div id="cart-modal">
  <div style="background:white; padding:20px; border-radius:10px; width:90%; max-width:480px; max-height:80vh; overflow-y:auto; position:relative;">
    <button onclick="closeCart()" style="position:absolute; top:10px; right:10px; cursor:pointer;">✖️</button>
    <h3>Корзина</h3>
    <ul id="cart-items" style="list-style:none; padding-left:0;"></ul>
    <p><strong>Итого: <span id="cart-total">0</span> сум</strong></p>
    <div style="display:flex; gap:10px;">
      <button onclick="clearCart()" class="btn" style="background:#c0392b;">Очистить корзину</button>
      <button onclick="checkout()" class="btn">Оформить заказ</button>
    </div>
    <p class="small">После нажатия «Оформить заказ» заказ отправится в Telegram и вам придёт уведомление.</p>
  </div>
</div>

<footer>
    &copy; 2025 Nobel_UZ. Все права защищены.
</footer>

<script>
    // Пример товаров — заменяй/добавляй товары здесь
    const PRODUCTS = [
        { id: 'p1', name: 'Товар 1', price: 10000, img: 'images/product1.jpg' },
        { id: 'p2', name: 'Товар 2', price: 15000, img: 'images/product2.jpg' },
        { id: 'p3', name: 'Товар 3', price: '20000', img: 'images/product3.jpg' }
    ];

    let cart = [];

    function loadCart() {
        const savedCart = localStorage.getItem('cart');
        if (savedCart) cart = JSON.parse(savedCart);
    }

    function saveCart() {
        localStorage.setItem('cart', JSON.stringify(cart));
    }

    function formatPrice(price) {
        return Number(price).toLocaleString('ru-RU');
    }

    function renderProducts() {
        const container = document.getElementById('products');
        container.innerHTML = '';
        PRODUCTS.forEach(p => {
            const div = document.createElement('div');
            div.className = 'product';
            div.innerHTML = `
                <img src="${p.img}" alt="${p.name}" loading="lazy" />
                <h3>${p.name}</h3>
                <p>Цена: ${formatPrice(p.price)} сум</p>
                <button class="btn">Добавить в корзину</button>
            `;
            const btn = div.querySelector('button');
            btn.addEventListener('click', () => addToCart(p));
            container.appendChild(div);
        });
    }

    function renderCart() {
        const cartItemsElem = document.getElementById('cart-items');
        const cartTotalElem = document.getElementById('cart-total');
        cartItemsElem.innerHTML = '';

        if (cart.length === 0) {
            cartItemsElem.innerHTML = '<li>Ваша корзина пуста.</li>';
            cartTotalElem.textContent = '0';
            return;
        }

        let total = 0;

        cart.forEach((item, index) => {
            const itemTotal = item.price * item.quantity;
            total += itemTotal;

            const li = document.createElement('li');
            li.style.marginBottom = '10px';
            li.className = 'cart-item';
            li.innerHTML = `
                <img src="${item.img}" alt="${item.name}" />
                <div style="flex:1;">
                  <strong>${item.name}</strong><br>
                  ${formatPrice(item.price)} сум ×
                  <button style="margin:0 5px; cursor:pointer;" onclick="decreaseQuantity(${index})">−</button>
                  ${item.quantity}
                  <button style="margin:0 5px; cursor:pointer;" onclick="increaseQuantity(${index})">+</button>
                  = <strong>${formatPrice(itemTotal)}</strong> сум
                </div>
                <button style="margin-left:10px; color:red; cursor:pointer;" onclick="removeFromCart(${index})">🗑</button>
            `;
            cartItemsElem.appendChild(li);
        });

        cartTotalElem.textContent = formatPrice(total);
    }

    function increaseQuantity(index) {
        cart[index].quantity += 1;
        saveCart();
        updateCartCount();
        renderCart();
    }

    function decreaseQuantity(index) {
        if (cart[index].quantity > 1) cart[index].quantity -= 1;
        else cart.splice(index, 1);
        saveCart();
        updateCartCount();
        renderCart();
    }

    function removeFromCart(index) {
        cart.splice(index, 1);
        updateCartCount();
        saveCart();
        renderCart();
    }

    function showCart() {
        renderCart();
        document.getElementById('cart-modal').classList.add('show');
    }

    function closeCart() {
        document.getElementById('cart-modal').classList.remove('show');
    }

    function clearCart() {
        cart = [];
        updateCartCount();
        saveCart();
        renderCart();
    }

    function addToCart(product) {
        const existing = cart.find(i => i.id === product.id);
        if (existing) existing.quantity += 1;
        else cart.push({ id: product.id, name: product.name, price: Number(product.price), quantity: 1, img: product.img });
        saveCart();
        updateCartCount();
        alert(product.name + ' добавлен в корзину');
    }

    function updateCartCount() {
        const totalItems = cart.reduce((s, it) => s + it.quantity, 0);
        document.getElementById('cart-count').textContent = totalItems;
    }

    async function checkout() {
        if (cart.length === 0) { alert('Корзина пуста'); return; }
        const name = prompt('Введите ваше имя');
        if (!name) { alert('Имя обязательно'); return; }
        const phone = prompt('Введите телефон (например +998...)');
        if (!phone) { alert('Телефон обязателен'); return; }

        const order = {
            customer: { name, phone },
            items: cart,
            total: cart.reduce((s,i)=>s+i.price*i.quantity,0),
            createdAt: new Date().toISOString()
        };

        // Отправляем заказ на serverless-функцию (Netlify)
        try {
            const res = await fetch('/.netlify/functions/sendOrder', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(order)
            });
            const data = await res.json();
            if (res.ok) {
                alert('Заказ отправлен! Мы свяжемся с вами.');
                clearCart();
                closeCart();
            } else {
                console.error(data);
                alert('Ошибка при отправке заказа. Попробуйте позже.');
            }
        } catch (err) {
            console.error(err);
            alert('Ошибка сети. Попробуйте позже.');
        }
    }

    // Инициализация
    loadCart();
    renderProducts();
    updateCartCount();
</script>

</body>
</html>
