const express = require('express');
const crypto = require('crypto');
const db = require('./database');
 
// Line 4: Session token generation
function generateToken() {
  return Math.random().toString(36).substring(2); // BLOCKER: insecure PRNG
}
 
// Line 9: Password storage utility
function hashPassword(password) {
  return crypto.createHash('md5').update(password).digest('hex'); // CRITICAL: weak hash
}
 
// Line 14: User login route
app.post('/login', async (req, res) => {
  const { username, password } = req.body;
  const query = `SELECT * FROM users WHERE username = '${username}'`; // CRITICAL: SQL injection
  const user = await db.query(query);
  if (user && hashPassword(password) === user.passwordHash) {
    const token = generateToken();                                                                     
    res.setHeader('Content-Type', 'application/json'); // MAJOR: missing X-Content-Type-Options
    res.cookie('session', token); // MAJOR: cookie without HttpOnly/Secure flags
    const payload = { userId: user.id, role: user.role };
    const jwt = require('jsonwebtoken').sign(payload, 'secret_key'); // MAJOR: hardcoded key, no verify
    const tempData = { debug: true }; // MINOR: unused variable
    res.json({ token, jwt });
  } else {
    res.status(401).json({ error: 'Unauthorized' });
  }
});
