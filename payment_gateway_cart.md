### Report on Payment Gateway Integration and Cart Page Development

---

## Table of Contents

1. [Introduction](#introduction)
2. [Project Overview](#project-overview)
3. [Technology Stack](#technology-stack)
4. [System Architecture](#system-architecture)
5. [Cart Page Development](#cart-page-development)
    - [Backend Implementation](#backend-implementation)
    - [Frontend Implementation](#frontend-implementation)
6. [Payment Gateway Integration](#payment-gateway-integration)
    - [Payment Gateway Selection](#payment-gateway-selection)
    - [Backend Integration](#backend-integration)
    - [Frontend Integration](#frontend-integration)
7. [Conclusion](#conclusion)

---

## 1. Introduction

This report details the development of two critical components in an e-commerce application: the Cart Page and Payment Gateway Integration. The Cart Page enables users to manage their selected products before proceeding to checkout, while the Payment Gateway Integration facilitates secure online payments. Both components are essential for providing a seamless shopping experience.

## 2. Project Overview

The project aims to implement a functional Cart Page and integrate a Payment Gateway into a React-based e-commerce application. Key features include:

- **Cart Page**: Adding, removing, and updating items in the cart.
- **Payment Gateway**: Secure processing of payments using a third-party service.

## 3. Technology Stack

- **Frontend**: React.js, Axios
- **Backend**: Node.js, Express.js
- **Database**: MongoDB
- **Payment Gateway**: Razorpay (or Stripe/PayPal)
- **Additional Libraries**: body-parser, cors, bcrypt.js, jsonwebtoken

## 4. System Architecture

The system architecture follows the Model-View-Controller (MVC) pattern:

- **Model**: Defines the data structure and interacts with the database.
- **View**: Represents the user interface.
- **Controller**: Handles requests, processes data, and returns responses.

---

## 5. Cart Page Development

### Backend Implementation

The backend handles the management of cart items, including adding, updating, and removing products. A `Cart` model was created to store cart data in MongoDB.

#### Cart Model

```javascript
const mongoose = require('mongoose');

const CartSchema = new mongoose.Schema({
    userId: { type: mongoose.Schema.Types.ObjectId, required: true, ref: 'User' },
    items: [
        {
            productId: { type: mongoose.Schema.Types.ObjectId, ref: 'Product', required: true },
            quantity: { type: Number, required: true, min: 1 },
        },
    ],
    totalAmount: { type: Number, required: true }
});

module.exports = mongoose.model('Cart', CartSchema);
```

#### Cart Routes

```javascript
const express = require('express');
const router = express.Router();
const Cart = require('../models/Cart');

// Get Cart by User ID
router.get('/:userId', async (req, res) => {
    try {
        const cart = await Cart.findOne({ userId: req.params.userId }).populate('items.productId');
        res.json(cart);
    } catch (error) {
        res.status(500).send('Server Error');
    }
});

// Add Item to Cart
router.post('/add', async (req, res) => {
    const { userId, productId, quantity } = req.body;

    try {
        let cart = await Cart.findOne({ userId });

        if (cart) {
            // Update existing cart
            const itemIndex = cart.items.findIndex(item => item.productId == productId);

            if (itemIndex > -1) {
                cart.items[itemIndex].quantity += quantity;
            } else {
                cart.items.push({ productId, quantity });
            }
        } else {
            // Create new cart
            cart = new Cart({ userId, items: [{ productId, quantity }], totalAmount: 0 });
        }

        cart.totalAmount = cart.items.reduce((total, item) => total + item.quantity * item.productId.price, 0);

        await cart.save();
        res.json(cart);
    } catch (error) {
        res.status(500).send('Server Error');
    }
});

// Remove Item from Cart
router.post('/remove', async (req, res) => {
    const { userId, productId } = req.body;

    try {
        let cart = await Cart.findOne({ userId });

        if (cart) {
            cart.items = cart.items.filter(item => item.productId != productId);
            cart.totalAmount = cart.items.reduce((total, item) => total + item.quantity * item.productId.price, 0);

            await cart.save();
            res.json(cart);
        } else {
            res.status(404).json({ msg: 'Cart not found' });
        }
    } catch (error) {
        res.status(500).send('Server Error');
    }
});

module.exports = router;
```

### Frontend Implementation

The frontend provides an interactive interface for users to manage their cart.

#### CartPage Component

```javascript
import React, { useState, useEffect } from 'react';
import axios from 'axios';

function CartPage({ userId }) {
    const [cart, setCart] = useState(null);

    useEffect(() => {
        // Fetch Cart by User ID
        axios.get(`/api/cart/${userId}`)
            .then(response => setCart(response.data))
            .catch(error => console.error('Error fetching cart', error));
    }, [userId]);

    const handleAddToCart = (productId, quantity) => {
        axios.post('/api/cart/add', { userId, productId, quantity })
            .then(response => setCart(response.data))
            .catch(error => console.error('Error adding to cart', error));
    };

    const handleRemoveFromCart = (productId) => {
        axios.post('/api/cart/remove', { userId, productId })
            .then(response => setCart(response.data))
            .catch(error => console.error('Error removing from cart', error));
    };

    if (!cart) return <p>Loading...</p>;

    return (
        <div>
            <h2>Your Cart</h2>
            <ul>
                {cart.items.map(item => (
                    <li key={item.productId._id}>
                        <p>{item.productId.name}</p>
                        <p>Quantity: {item.quantity}</p>
                        <p>Price: ${item.productId.price}</p>
                        <button onClick={() => handleRemoveFromCart(item.productId._id)}>Remove</button>
                    </li>
                ))}
            </ul>
            <h3>Total: ${cart.totalAmount}</h3>
        </div>
    );
}

export default CartPage;
```

---

## 6. Payment Gateway Integration

### Payment Gateway Selection

For this project, Razorpay was chosen due to its comprehensive features, ease of integration, and support for various payment methods. Alternatives like Stripe and PayPal were also considered.

### Backend Integration

The backend handles payment processing by interacting with the Razorpay API to create payment orders and verify transactions.

#### Payment Routes

```javascript
const express = require('express');
const router = express.Router();
const Razorpay = require('razorpay');
const crypto = require('crypto');

const razorpay = new Razorpay({
    key_id: 'YOUR_RAZORPAY_KEY_ID',
    key_secret: 'YOUR_RAZORPAY_SECRET'
});

router.post('/order', async (req, res) => {
    const { amount, currency, receipt } = req.body;

    try {
        const options = { amount: amount * 100, currency, receipt };
        const order = await razorpay.orders.create(options);
        res.json(order);
    } catch (error) {
        res.status(500).send('Server Error');
    }
});

router.post('/verify', async (req, res) => {
    const { razorpay_order_id, razorpay_payment_id, razorpay_signature } = req.body;

    const hmac = crypto.createHmac('sha256', 'YOUR_RAZORPAY_SECRET');
    hmac.update(razorpay_order_id + "|" + razorpay_payment_id);
    const generatedSignature = hmac.digest('hex');

    if (generatedSignature === razorpay_signature) {
        res.json({ msg: 'Payment verified' });
    } else {
        res.status(400).json({ msg: 'Payment verification failed' });
    }
});

module.exports = router;
```

### Frontend Integration

The frontend includes a PaymentPage component that allows users to make payments via Razorpay.

#### PaymentPage Component

```javascript
import React from 'react';
import axios from 'axios';
import Razorpay from 'razorpay';

function PaymentPage({ amount, currency, userId }) {
    const handlePayment = async () => {
        try {
            const order = await axios.post('/api/payment/order', { amount, currency, receipt: `receipt_${userId}` });
            const options = {
                key: 'YOUR_RAZORPAY_KEY_ID',
                amount: order.data.amount,
                currency: order.data.currency,
                name: 'Your Shop',
                description: 'Test Transaction',
                order_id: order.data.id,
                handler: async (response) => {
                    await axios.post('/api/payment/verify', response);
                    alert('Payment successful');
                }
            };

            const rzp = new Razorpay(options);
            rzp.open();
        } catch (error) {
            alert('Error during payment');
        }
    };

    return (
        <div>
            <h2>Total Amount: ${amount}</h2>
            <button onClick={handlePayment}>Pay Now</button>
        </div>
    );


}

export default PaymentPage;
```

---

## 7. Conclusion

The Cart Page and Payment Gateway Integration are integral parts of any e-commerce application, providing users with essential functionality for managing their shopping experience. The backend and frontend implementations ensure that the system is both efficient and secure, offering a seamless and reliable shopping experience. This project demonstrates the effective use of modern technologies and best practices in web development.

---

This concludes the report on Payment Gateway Integration and Cart Page development. The code snippets and descriptions provided should serve as a comprehensive guide for understanding and implementing these features in a React-based e-commerce application.
