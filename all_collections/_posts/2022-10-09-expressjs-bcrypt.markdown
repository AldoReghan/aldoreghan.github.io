---
layout: post
title: ExpressJS Auth with bcrypt
date: 2022-10-09
categories: ["ExpressJS"]
---

Halo pada tulisan ini saya akan implementasi auth menggunakan bcrypt, alasan saya membuat tulisan ini, karena banyak sekali resource yang saya pelajari rata-rata memiliki masalah saat melakukan compare saat implementasi auth yang dimana datanya di fetch dari database.

1. Menambahkan bcrypt pada project kita.<br>
   ```npm
   npm i bcrypt
   ```
2. mengimport bcrypt<br>
   ```js
   var bcrypt = require("bcrypt");
   ```
3. Membuat sebuah function register yang dimana saat melakukan register, password yang kita buat akan dienkripsi oleh bcrypt
   <br><br>
   **app.js**
   <br><br>
   ```js
    app.post('/register', async (req, res, next) => {
  
    const username = req.body.username
    const passwords = req.body.password
  
    if(username && passwords){
        if (username.length >= 6 && passwords.length >= 6) {
        var salt = await bcrypt.genSalt(10);
        const password = await bcrypt.hash(passwords, salt);
        conn.query("SELECT * FROM users WHERE username = ?", [username], function (err, rows, fields) {
            if (rows.length > 0) {
            res.status(400).send({ message: 'Username already exists' })
            } else {
            const token = uuidv4();
            conn.query('INSERT INTO users(username, password, token ,point, status) VALUES(?,?,?,?,?)', [username, password, token, 10, 1], function (error, results) {
                res.status(200).send({ message: 'Register successfully' })
            });
            }
        })
        } else {
        res.status(400).send({ message: 'Username and Password should be at least 6 character' });
        }
    }else{
        res.status(500).send({message: "Something was missing!"});
    }

    });
   ```
4. Membuat function login, yang dimana akan kita implementasikan, melakukan compare password yang akan kita inputkan dengan password yang telah di dienkripsi sebelumnya
   <br><br>
   **app.js**
   <br><br>
   ```js
   app.post('/login', async (req, res) => {
    const username = req.body.username;
    const password = req.body.password;
    if (username && password) {
        conn.query('SELECT * FROM users WHERE username = ?', [username], function (error, rows, fields) {
        if (rows.length == 0) {
            res.status(400).send({ message: 'Login failed'})
        } else {
            if(rows[0]['status'] == 0){
            res.status(500).send({message: 'Login Failed, your account is inactive'});
            return false;
            }
            var compare = bcrypt.compareSync(password, rows[0]['password'])
            if (compare == true) {
            res.status(200).send({ message: 'Login successfully', token: rows[0]['token'] })
            } else {
            res.status(400).send({ message: 'Login failed' })
            }
        }
        });
    } else {
        res.status(400).send({ message: 'Please enter username and password' })
    }
    });
   ```