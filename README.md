// app.js
const express = require('express');
const { MongoClient, ObjectId } = require('mongodb');
const NodeCache = require('node-cache');

const app = express();
const port = 3000;

app.use(express.json());

// ====== MongoDB setup ======
const url = 'mongodb://localhost:27017';
const dbName = 'lab7_db';
let db;

MongoClient.connect(url, { useUnifiedTopology: true })
    .then(client => {
        db = client.db(dbName);
        console.log('MongoDB connected!');
    })
    .catch(err => console.error(err));

// ====== Кеш ======
const cache = new NodeCache({ stdTTL: 60 }); // 1 хв кеш

// ====== REST API ======

// Створити пост
app.post('/posts', async (req, res) => {
    const { title, content, tags } = req.body;
    const post = { title, content, tags, likes: 0, dislikes: 0, createdAt: new Date() };
    const result = await db.collection('posts').insertOne(post);
    res.json(result.ops[0]);
});

// Отримати всі пости з пагінацією
app.get('/posts', async (req, res) => {
    const page = parseInt(req.query.page) || 1;
    const limit = parseInt(req.query.limit) || 5;
    const skip = (page - 1) * limit;

    const cacheKey = `posts_page_${page}_limit_${limit}`;
    if (cache.has(cacheKey)) {
        return res.json({ cached: true, data: cache.get(cacheKey) });
    }

    const posts = await db.collection('posts')
        .find({})
        .sort({ createdAt: -1 })
        .skip(skip)
        .limit(limit)
        .toArray();

    cache.set(cacheKey, posts);
    res.json({ cached: false, data: posts });
});

// Пошук по тегу
app.get('/posts/search', async (req, res) => {
    const { tag } = req.query;
    const posts = await db.collection('posts')
        .find({ tags: tag })
        .toArray();
    res.json(posts);
});

// Лайк/дизлайк
app.post('/posts/:id/like', async (req, res) => {
    const id = req.params.id;
    const result = await db.collection('posts')
        .findOneAndUpdate({ _id: ObjectId(id) }, { $inc: { likes: 1 } }, { returnDocument: 'after' });
    res.json(result.value);
});

app.post('/posts/:id/dislike', async (req, res) => {
    const id = req.params.id;
    const result = await db.collection('posts')
        .findOneAndUpdate({ _id: ObjectId(id) }, { $inc: { dislikes: 1 } }, { returnDocument: 'after' });
    res.json(result.value);
});

// Аналітика
app.get('/analytics', async (req, res) => {
    const totalPosts = await db.collection('posts').countDocuments();
    const totalLikes = await db.collection('posts').aggregate([{ $group: { _id: null, sum: { $sum: "$likes" }}}]).toArray();
    const totalDislikes = await db.collection('posts').aggregate([{ $group: { _id: null, sum: { $sum: "$dislikes" }}}]).toArray();

    res.json({
        totalPosts,
        totalLikes: totalLikes[0]?.sum || 0,
        totalDislikes: totalDislikes[0]?.sum || 0
    });
});

// ====== Запуск сервера ======
app.listen(port, () => {
    console.log(`Server running at http://localhost:${port}`);
});

// app.js
const express = require('express');
const { MongoClient, ObjectId } = require('mongodb');
const NodeCache = require('node-cache');

const app = express();
const port = 3000;

app.use(express.json());

// ====== MongoDB setup ======
const url = 'mongodb://localhost:27017';
const dbName = 'lab7_db';
let db;

MongoClient.connect(url, { useUnifiedTopology: true })
    .then(client => {
        db = client.db(dbName);
        console.log('MongoDB connected!');
    })
    .catch(err => console.error(err));

// ====== Кеш ======
const cache = new NodeCache({ stdTTL: 60 }); // 1 хв кеш

// ====== REST API ======

// Створити пост
app.post('/posts', async (req, res) => {
    const { title, content, tags } = req.body;
    const post = { title, content, tags, likes: 0, dislikes: 0, createdAt: new Date() };
    const result = await db.collection('posts').insertOne(post);
    res.json(result.ops[0]);
});

// Отримати всі пости з пагінацією
app.get('/posts', async (req, res) => {
    const page = parseInt(req.query.page) || 1;
    const limit = parseInt(req.query.limit) || 5;
    const skip = (page - 1) * limit;

    const cacheKey = `posts_page_${page}_limit_${limit}`;
    if (cache.has(cacheKey)) {
        return res.json({ cached: true, data: cache.get(cacheKey) });
    }

    const posts = await db.collection('posts')
        .find({})
        .sort({ createdAt: -1 })
        .skip(skip)
        .limit(limit)
        .toArray();

    cache.set(cacheKey, posts);
    res.json({ cached: false, data: posts });
});

// Пошук по тегу
app.get('/posts/search', async (req, res) => {
    const { tag } = req.query;
    const posts = await db.collection('posts')
        .find({ tags: tag })
        .toArray();
    res.json(posts);
});

// Лайк/дизлайк
app.post('/posts/:id/like', async (req, res) => {
    const id = req.params.id;
    const result = await db.collection('posts')
        .findOneAndUpdate({ _id: ObjectId(id) }, { $inc: { likes: 1 } }, { returnDocument: 'after' });
    res.json(result.value);
});

app.post('/posts/:id/dislike', async (req, res) => {
    const id = req.params.id;
    const result = await db.collection('posts')
        .findOneAndUpdate({ _id: ObjectId(id) }, { $inc: { dislikes: 1 } }, { returnDocument: 'after' });
    res.json(result.value);
});

// Аналітика
app.get('/analytics', async (req, res) => {
    const totalPosts = await db.collection('posts').countDocuments();
    const totalLikes = await db.collection('posts').aggregate([{ $group: { _id: null, sum: { $sum: "$likes" }}}]).toArray();
    const totalDislikes = await db.collection('posts').aggregate([{ $group: { _id: null, sum: { $sum: "$dislikes" }}}]).toArray();

    res.json({
        totalPosts,
        totalLikes: totalLikes[0]?.sum || 0,
        totalDislikes: totalDislikes[0]?.sum || 0
    });
});

// ====== Запуск сервера ======
app.listen(port, () => {
    console.log(`Server running at http://localhost:${port}`);
});

// app.js
const express = require('express');
const { MongoClient, ObjectId } = require('mongodb');
const NodeCache = require('node-cache');

const app = express();
const port = 3000;

app.use(express.json());

// ====== MongoDB setup ======
const url = 'mongodb://localhost:27017';
const dbName = 'lab7_db';
let db;

MongoClient.connect(url, { useUnifiedTopology: true })
    .then(client => {
        db = client.db(dbName);
        console.log('MongoDB connected!');
    })
    .catch(err => console.error(err));

// ====== Кеш ======
const cache = new NodeCache({ stdTTL: 60 }); // 1 хв кеш

// ====== REST API ======

// Створити пост
app.post('/posts', async (req, res) => {
    const { title, content, tags } = req.body;
    const post = { title, content, tags, likes: 0, dislikes: 0, createdAt: new Date() };
    const result = await db.collection('posts').insertOne(post);
    res.json(result.ops[0]);
});

// Отримати всі пости з пагінацією
app.get('/posts', async (req, res) => {
    const page = parseInt(req.query.page) || 1;
    const limit = parseInt(req.query.limit) || 5;
    const skip = (page - 1) * limit;

    const cacheKey = `posts_page_${page}_limit_${limit}`;
    if (cache.has(cacheKey)) {
        return res.json({ cached: true, data: cache.get(cacheKey) });
    }

    const posts = await db.collection('posts')
        .find({})
        .sort({ createdAt: -1 })
        .skip(skip)
        .limit(limit)
        .toArray();

    cache.set(cacheKey, posts);
    res.json({ cached: false, data: posts });
});

// Пошук по тегу
app.get('/posts/search', async (req, res) => {
    const { tag } = req.query;
    const posts = await db.collection('posts')
        .find({ tags: tag })
        .toArray();
    res.json(posts);
});

// Лайк/дизлайк
app.post('/posts/:id/like', async (req, res) => {
    const id = req.params.id;
    const result = await db.collection('posts')
        .findOneAndUpdate({ _id: ObjectId(id) }, { $inc: { likes: 1 } }, { returnDocument: 'after' });
    res.json(result.value);
});

app.post('/posts/:id/dislike', async (req, res) => {
    const id = req.params.id;
    const result = await db.collection('posts')
        .findOneAndUpdate({ _id: ObjectId(id) }, { $inc: { dislikes: 1 } }, { returnDocument: 'after' });
    res.json(result.value);
});

// Аналітика
app.get('/analytics', async (req, res) => {
    const totalPosts = await db.collection('posts').countDocuments();
    const totalLikes = await db.collection('posts').aggregate([{ $group: { _id: null, sum: { $sum: "$likes" }}}]).toArray();
    const totalDislikes = await db.collection('posts').aggregate([{ $group: { _id: null, sum: { $sum: "$dislikes" }}}]).toArray();

    res.json({
        totalPosts,
        totalLikes: totalLikes[0]?.sum || 0,
        totalDislikes: totalDislikes[0]?.sum || 0
    });
});

// ====== Запуск сервера ======
app.listen(port, () => {
    console.log(`Server running at http://localhost:${port}`);
});
