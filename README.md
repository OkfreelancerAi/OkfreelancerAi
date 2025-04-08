- 👋 Hi, I’m @OkfreelancerAi
-// Import necessary modules
const express = require('express');
const bodyParser = require('body-parser');
const mongoose = require('mongoose');
const bcrypt = require('bcrypt');
const jwt = require('jsonwebtoken');

const app = express();
app.use(bodyParser.json());

// MongoDB connection
mongoose.connect('mongodb://localhost:27017/adultContentApp', {
  useNewUrlParser: true,
  useUnifiedTopology: true,
});

// User schema for authentication and affiliate tracking
const userSchema = new mongoose.Schema({
  username: { type: String, required: true },
  password: { type: String, required: true },
  ageVerified: { type: Boolean, default: false },
  affiliateCode: { type: String, unique: true }, // Unique affiliate code
  referredBy: { type: String }, // Tracks who referred the user
  earnings: { type: Number, default: 0 }, // Earnings from referrals
});

userSchema.pre('save', async function (next) {
  if (this.isModified('password')) {
    this.password = await bcrypt.hash(this.password, 10);
  }
  next();
});

const User = mongoose.model('User', userSchema);

// Content schema for managing videos
const contentSchema = new mongoose.Schema({
  title: { type: String, required: true },
  category: { type: String, required: true },
  url: { type: String, required: true }, // Video URL
  price: { type: Number, required: true }, // Pay-to-watch price
});

const Content = mongoose.model('Content', contentSchema);

// Middleware for authentication
const authenticate = (req, res, next) => {
  const token = req.headers.authorization;
  if (!token) return res.status(401).send('Access denied');

  try {
    const decoded = jwt.verify(token, 'secretKey');
    req.user = decoded;
    next();
  } catch (err) {
    res.status(400).send('Invalid token');
  }
};

// Routes

// Register a new user
app.post('/register', async (req, res) => {
  const { username, password, referredBy } = req.body;

  try {
    const affiliateCode = `${username}-${Date.now()}`; // Generate unique affiliate code
    const user = new User({ username, password, affiliateCode, referredBy });
    await user.save();

    // Reward the referrer if applicable
    if (referredBy) {
      const referrer = await User.findOne({ affiliateCode: referredBy });
      if (referrer) {
        referrer.earnings += 10; // Example reward for referral
        await referrer.save();
      }
    }

    res.status(201).send({ message: 'User registered successfully', affiliateCode });
  } catch (err) {
    res.status(400).send(err.message);
  }
});

// Login a user
app.post('/login', async (req, res) => {
  const { username, password } = req.body;

  const user = await User.findOne({ username });
  if (!user) return res.status(404).send('User not found');

  const validPassword = await bcrypt.compare(password, user.password);
  if (!validPassword) return res.status(400).send('Invalid credentials');

  const token = jwt.sign({ id: user._id }, 'secretKey', { expiresIn: '1h' });
  res.send({ token });
});

// Add new content (Admin only)
app.post('/content', authenticate, async (req, res) => {
  const { title, category, url, price } = req.body;

  try {
    const content = new Content({ title, category, url, price });
    await content.save();
    res.status(201).send('Content added successfully');
  } catch (err) {
    res.status(400).send(err.message);
  }
});

// Get all content (Pay-to-watch model)
app.get('/content', authenticate, async (req, res) => {
  try {
    const contentList = await Content.find();
    res.send(contentList);
  } catch (err) {
    res.status(500).send(err.message);
  }
});

// Affiliate dashboard to check earnings
app.get('/affiliate-dashboard', authenticate, async (req, res) => {
  try {
    const user = await User.findById(req.user.id);
    if (!user) return res.status(404).send('User not found');

    res.send({
      username: user.username,
      affiliateCode: user.affiliateCode,
      earnings: user.earnings,
      referredBy: user.referredBy || 'None',
    });
  } catch (err) {
    res.status(500).send(err.message);
  }
});

// Start server
app.listen(3000, () => console.log('Server running on port 3000'));
// Import necessary modules
const express = require('express');
const bodyParser = require('body-parser');
const mongoose = require('mongoose');
const bcrypt = require('bcrypt');
const jwt = require('jsonwebtoken');

const app = express();
app.use(bodyParser.json());

// MongoDB connection
mongoose.connect('mongodb://localhost:27017/adultContentApp', {
  useNewUrlParser: true,
  useUnifiedTopology: true,
});

// User schema for authentication and affiliate tracking
const userSchema = new mongoose.Schema({
  username: { type: String, required: true },
  password: { type: String, required: true },
  ageVerified: { type: Boolean, default: false },
  affiliateCode: { type: String, unique: true }, // Unique affiliate code
  referredBy: { type: String }, // Tracks who referred the user
  earnings: { type: Number, default: 0 }, // Earnings from referrals
});

userSchema.pre('save', async function (next) {
  if (this.isModified('password')) {
    this.password = await bcrypt.hash(this.password, 10);
  }
  next();
});

const User = mongoose.model('User', userSchema);

// Content schema for managing videos
const contentSchema = new mongoose.Schema({
  title: { type: String, required: true },
  category: { type: String, required: true },
  url: { type: String, required: true }, // Video URL
  price: { type: Number, required: true }, // Pay-to-watch price
});

const Content = mongoose.model('Content', contentSchema);

// Middleware for authentication
const authenticate = (req, res, next) => {
  const token = req.headers.authorization;
  if (!token) return res.status(401).send('Access denied');

  try {
    const decoded = jwt.verify(token, 'secretKey');
    req.user = decoded;
    next();
  } catch (err) {
    res.status(400).send('Invalid token');
  }
};

// Routes

// Register a new user
app.post('/register', async (req, res) => {
  const { username, password, referredBy } = req.body;

  try {
    const affiliateCode = `${username}-${Date.now()}`; // Generate unique affiliate code
    const user = new User({ username, password, affiliateCode, referredBy });
    await user.save();

    // Reward the referrer if applicable
    if (referredBy) {
      const referrer = await User.findOne({ affiliateCode: referredBy });
      if (referrer) {
        referrer.earnings += 10; // Example reward for referral
        await referrer.save();
      }
    }

    res.status(201).send({ message: 'User registered successfully', affiliateCode });
  } catch (err) {
    res.status(400).send(err.message);
  }
});

// Login a user
app.post('/login', async (req, res) => {
  const { username, password } = req.body;

  const user = await User.findOne({ username });
  if (!user) return res.status(404).send('User not found');

  const validPassword = await bcrypt.compare(password, user.password);
  if (!validPassword) return res.status(400).send('Invalid credentials');

  const token = jwt.sign({ id: user._id }, 'secretKey', { expiresIn: '1h' });
  res.send({ token });
});

// Add new content (Admin only)
app.post('/content', authenticate, async (req, res) => {
  const { title, category, url, price } = req.body;

  try {
    const content = new Content({ title, category, url, price });
    await content.save();
    res.status(201).send('Content added successfully');
  } catch (err) {
    res.status(400).send(err.message);
  }
});

// Get all content (Pay-to-watch model)
app.get('/content', authenticate, async (req, res) => {
  try {
    const contentList = await Content.find();
    res.send(contentList);
  } catch (err) {
    res.status(500).send(err.message);
  }
});

// Affiliate dashboard to check earnings
app.get('/affiliate-dashboard', authenticate, async (req, res) => {
  try {
    const user = await User.findById(req.user.id);
    if (!user) return res.status(404).send('User not found');

    res.send({
      username: user.username,
      affiliateCode: user.affiliateCode,
      earnings: user.earnings,
      referredBy: user.referredBy || 'None',
    });
  } catch (err) {
    res.status(500).send(err.message);
  }
});

// Start server
app.listen(3000, () => console.log('Server running on port 3000'));
![1000003355](https://github.com/user-attachments/assets/367f1ec9-93a1-472a-84bb-78a87a4fc614)
![1000003356](https://github.com/user-attachments/assets/f19f1501-667d-415d-beb5-b7f29f579864)
![1000003357](https://github.com/user-attachments/assets/da130b7c-116f-4abb-ac04-15920d115c5a)
![1000003358](https://github.com/user-attachments/assets/12fbee1e-effa-48b7-8b3f-26df535186d6)
![1000003360](https://github.com/user-attachments/assets/06970ea9-2ba0-4c30-98f9-f6a6c9a79676)
![1000003361](https://github.com/user-attachments/assets/5440523b-dd38-480e-9d79-fa3d26a38d11)
![1000003751](https://github.com/user-attachments/assets/6ccc7015-5ffc-4854-9af9-7eb06953b420)
![1000003752](https://github.com/user-attachments/assets/082322b0-c045-4ca8-a4fc-e2742e499606)
![1000003753](https://github.com/user-attachments/assets/d15f9e2f-298e-44da-8e37-36212a509f18)
![1000003754](https://github.com/user-attachments/assets/b39067c6-0d9a-497a-a821-68733f03d31b)
![1000003758](https://github.com/user-attachments/assets/a09ddc24-99d6-4ff5-9e72-2185f626aeac)
I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...

<!---
OkfreelancerAi/OkfreelancerAi is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
