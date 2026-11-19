# onlinefoodordersystem
Online Food Ordering System – Description An Online Food Ordering System is a web-based application 
vOnline Food Ordering System – Project Description
1. Introduction
The Online Food Ordering System is a web-based application that allows customers to order food from restaurants through the internet. Customers can view food items, check prices, select their favourite dishes, add items to the cart, and place orders online without visiting the restaurant.

The system provides an easy and convenient way for customers to order food. It also helps restaurant owners manage food items, customer orders, payments, and delivery details efficiently.

2. Objectives
To provide an easy and user-friendly food ordering platform.

To allow customers to browse different food items and menus.

To reduce the time required to order food.

To provide online order placement and payment facilities.

To help restaurant administrators manage food items and orders.
food_order/
│
├── index.php
├── style.css
├── db.php
├── cart.js
├── order.php
└── database.sql
vvvvCREATE DATABASE food_order;
USE food_order;

CREATE TABLE foods (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100),
    price DECIMAL(10,2),
    image VARCHAR(255)
);

CREATE TABLE orders (
    id INT AUTO_INCREMENT PRIMARY KEY,
    customer_name VARCHAR(100),
    phone VARCHAR(15),
    address TEXT,
    total DECIMAL(10,2),
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE order_items (
    id INT AUTO_INCREMENT PRIMARY KEY,
    order_id INT,
    food_name VARCHAR(100),
    quantity INT,
    price DECIMAL(10,2)
);

INSERT INTO foods (name, price, image) VALUES
('Pizza', 250, 'pizza.jpg'),
('Burger', 120, 'burger.jpg'),
('Biryani', 180, 'biryani.jpg'),
('Dosa', 60, 'dosa.jpg'),
('Fried Rice', 150, 'rice.jpg'),
('Sandwich', 90, 'sandwich.jpg');<?php
$conn = new mysqli("localhost", "root", "", "food_order");

if ($conn->connect_error) {
    die("Database connection failed");
}
?><?php
include 'db.php';
$result = $conn->query("SELECT * FROM foods");
?>

<!DOCTYPE html>
<html>
<head>
    <title>Online Food Ordering System</title>
    <link rel="stylesheet" href="style.css">
</head>

<body>

<header>
    <h1>🍔 Online Food Ordering</h1>
    <p>Delicious Food Delivered To Your Door</p>
</header>

<h2>Our Food Menu</h2>

<div class="menu">

<?php while($food = $result->fetch_assoc()) { ?>

<div class="food-card">

    <img src="<?php echo htmlspecialchars($food['image']); ?>">

    <h3><?php echo htmlspecialchars($food['name']); ?></h3>

    <p>₹<?php echo $food['price']; ?></p>

    <button onclick="addToCart(
        '<?php echo htmlspecialchars(addslashes($food['name']), ENT_QUOTES); ?>',
        <?php echo $food['price']; ?>
    )">
        Add to Cart
    </button>

</div>

<?php } ?>

</div>

<h2>Shopping Cart</h2>

<div id="cart"></div>

<h3 id="total">Total: ₹0</h3>

<form action="order.php" method="POST" onsubmit="return checkout()">

    <input type="text" name="customer_name"
        placeholder="Enter Your Name" required>

    <input type="text" name="phone"
        placeholder="Enter Phone Number" required>

    <textarea name="address"
        placeholder="Enter Delivery Address" required></textarea>

    <input type="hidden" name="cart_data" id="cart_data">

    <button type="submit">Place Order</button>

</form>

<script src="cart.js"></script>

</body>
</html>vbody {
    margin: 0;
    font-family: Arial, sans-serif;
    background: #f5f5f5;
    text-align: center;
}

header {
    background: #ff5722;
    color: white;
    padding: 25px;
}

h2 {
    color: #333;
    margin-top: 30px;
}

.menu {
    display: flex;
    justify-content: center;
    flex-wrap: wrap;
    gap: 20px;
    padding: 20px;
}

.food-card {
    background: white;
    width: 200px;
    padding: 15px;
    border-radius: 10px;
    box-shadow: 0 2px 8px #ccc;
}

.food-card img {
    width: 100%;
    height: 130px;
    object-fit: cover;
    border-radius: 8px;
}

button {
    background: #ff5722;
    color: white;
    border: none;
    padding: 10px 20px;
    cursor: pointer;
    border-radius: 5px;
}

button:hover {
    background: #e64a19;
}

form {
    background: white;
    padding: 25px;
    margin: 30px auto;
    max-width: 400px;
    border-radius: 10px;
}

input, textarea {
    width: 90%;
    padding: 12px;
    margin: 10px;
    border: 1px solid #ccc;
    border-radius: 5px;
}

#cart {
    margin: 20px;
}let cart = [];

function addToCart(name, price) {

    let item = cart.find(x => x.name === name);

    if (item) {
        item.quantity++;
    } else {
        cart.push({
            name: name,
            price: price,
            quantity: 1
        });
    }

    displayCart();
}

function displayCart() {

    let output = "";
    let total = 0;

    cart.forEach((item, index) => {

        let subtotal = item.price * item.quantity;
        total += subtotal;

        output += `
            <p>
                ${item.name} - ₹${item.price}
                <button onclick="changeQuantity(${index}, -1)">-</button>
                ${item.quantity}
                <button onclick="changeQuantity(${index}, 1)">+</button>
                ₹${subtotal}
            </p>
        `;
    });

    document.getElementById("cart").innerHTML = output;
    document.getElementById("total").innerHTML =
        "Total: ₹" + total;
}

function changeQuantity(index, amount) {

    cart[index].quantity += amount;

    if (cart[index].quantity <= 0) {
        cart.splice(index, 1);
    }

    displayCart();
}

function checkout() {

    if (cart.length === 0) {
        alert("Please add food items to your cart!");
        return false;
    }

    document.getElementById("cart_data").value =
        JSON.stringify(cart);

    return true;
}v<?php
include 'db.php';

if ($_SERVER["REQUEST_METHOD"] == "POST") {

    $name = $_POST['customer_name'];
    $phone = $_POST['phone'];
    $address = $_POST['address'];

    $cart = json_decode($_POST['cart_data'], true);

    if (!is_array($cart) || empty($cart)) {
        die("Invalid cart");
    }

    $total = 0;

    foreach ($cart as $item) {
        if (!isset($item['name'], $item['price'], $item['quantity'])
            || !is_numeric($item['price'])
            || !is_numeric($item['quantity'])
            || $item['price'] < 0
            || $item['quantity'] < 1) {
            die("Invalid order item");
        }

        $total += $item['price'] * $item['quantity'];
    }

    $stmt = $conn->prepare(
        "INSERT INTO orders
        (customer_name, phone, address, total)
        VALUES (?, ?, ?, ?)"
    );

    $stmt->bind_param("sssd", $name, $phone, $address, $total);
    $stmt->execute();

    $order_id = $conn->insert_id;

    $itemStmt = $conn->prepare(
        "INSERT INTO order_items
        (order_id, food_name, quantity, price)
        VALUES (?, ?, ?, ?)"
    );

    foreach ($cart as $item) {

        $food = $item['name'];
        $quantity = (int)$item['quantity'];
        $price = (float)$item['price'];

        $itemStmt->bind_param(
            "isid", $order_id, $food, $quantity, $price
        );

        $itemStmt->execute();
    }

    echo "<h1>Order Placed Successfully!</h1>";
    echo "<h3>Your Order ID: " . $order_id . "</h3>";
    echo "<p>Total Amount: ₹" . $total . "</p>";
    echo "<a href='index.php'>Back to Home</a>";

} else {
    echo "Invalid request";
}
?>
