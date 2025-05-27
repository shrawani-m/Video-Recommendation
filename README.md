# Video Recommendation Website - Full Stack Implementation

This codebase includes both frontend (React) and backend (Node.js/Express) components for the video recommendation website. The project is structured as follows:

## Project Structure


video-recommender/
├── frontend/                # React frontend application
│   ├── public/              # Static files
│   └── src/                 # React source code
│       ├── components/      # UI components
│       ├── pages/           # Page components
│       ├── styles/          # CSS files
│       ├── utils/           # Utility functions
│       ├── App.js           # Main App component
│       └── index.js         # Entry point
│
└── backend/                 # Node.js/Express backend
    ├── config/              # Configuration files
    ├── controllers/         # Route controllers
    ├── models/              # Database models
    ├── routes/              # API routes
    ├── services/            # Business logic
    ├── utils/               # Utility functions
    ├── app.js               # Express app setup
    └── server.js            # Entry point


## Frontend Code (React)

### frontend/src/index.js - React Entry Point

javascript
import React from 'react';
import ReactDOM from 'react-dom/client';
import './styles/index.css';
import App from './App';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);


### frontend/src/App.js - Main App Component

javascript
import React from 'react';
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';
import Header from './components/Header';
import Footer from './components/Footer';
import HomePage from './pages/HomePage';
import RecommendationsPage from './pages/RecommendationsPage';
import './styles/App.css';

function App() {
  return (
    <Router>
      <div className="app">
        <Header />
        <main className="main-content">
          <Routes>
            <Route path="/" element={<HomePage />} />
            <Route path="/recommendations" element={<RecommendationsPage />} />
          </Routes>
        </main>
        <Footer />
      </div>
    </Router>
  );
}

export default App;


### frontend/src/components/Header.js - Header Component

javascript
import React from 'react';
import { Link } from 'react-router-dom';
import '../styles/Header.css';

function Header() {
  return (
    <header className="header">
      <div className="logo">
        <Link to="/">BreakTime Videos</Link>
      </div>
      <nav className="nav">
        <ul>
          <li><Link to="/">Home</Link></li>
          <li><a href="https://github.com/yourusername/video-recommender" target="_blank" rel="noopener noreferrer">GitHub</a></li>
        </ul>
      </nav>
    </header>
  );
}

export default Header;


### frontend/src/components/Footer.js - Footer Component

javascript
import React from 'react';
import '../styles/Footer.css';

function Footer() {
  return (
    <footer className="footer">
      <p>&copy; {new Date().getFullYear()} BreakTime Videos | All Rights Reserved</p>
    </footer>
  );
}

export default Footer;


### frontend/src/pages/HomePage.js - Home/Preferences Page

javascript
import React, { useState } from 'react';
import { useNavigate } from 'react-router-dom';
import '../styles/HomePage.css';

function HomePage() {
  const navigate = useNavigate();
  const [preferences, setPreferences] = useState({
    mood: 'happy',
    interests: 'comedy',
    timeAvailable: '20',
  });

  const moods = ['happy', 'relaxed', 'curious', 'energetic', 'focused'];
  const interestCategories = [
    'comedy', 'education', 'science', 'cooking', 'fitness',
    'technology', 'music', 'travel', 'gaming', 'art'
  ];
  const timeOptions = ['20', '25', '30'];

  const handleChange = (e) => {
    const { name, value } = e.target;
    setPreferences(prev => ({
      ...prev,
      [name]: value
    }));
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    
    // Store preferences in localStorage for persistence
    localStorage.setItem('videoPreferences', JSON.stringify(preferences));
    
    // Navigate to recommendations page with query params
    navigate(`/recommendations?mood=${preferences.mood}&interests=${preferences.interests}&time=${preferences.timeAvailable}`);
  };

  return (
    <div className="home-container">
      <div className="hero">
        <h1>Find the Perfect Video for Your Break</h1>
        <p>Tell us how you're feeling, what you're interested in, and how much time you have.</p>
      </div>
      
      <form className="preferences-form" onSubmit={handleSubmit}>
        <div className="form-group">
          <label htmlFor="mood">How are you feeling?</label>
          <select 
            id="mood" 
            name="mood" 
            value={preferences.mood} 
            onChange={handleChange}
          >
            {moods.map(mood => (
              <option key={mood} value={mood}>{mood.charAt(0).toUpperCase() + mood.slice(1)}</option>
            ))}
          </select>
        </div>

        <div className="form-group">
          <label htmlFor="interests">What are you interested in?</label>
          <select 
            id="interests" 
            name="interests" 
            value={preferences.interests} 
            onChange={handleChange}
          >
            {interestCategories.map(category => (
              <option key={category} value={category}>{category.charAt(0).toUpperCase() + category.slice(1)}</option>
            ))}
          </select>
        </div>

        <div className="form-group">
          <label htmlFor="timeAvailable">How much time do you have?</label>
          <select 
            id="timeAvailable" 
            name="timeAvailable" 
            value={preferences.timeAvailable} 
            onChange={handleChange}
          >
            {timeOptions.map(time => (
              <option key={time} value={time}>{time} minutes</option>
            ))}
          </select>
        </div>

        <button type="submit" className="submit-btn">Find Videos</button>
      </form>
    </div>
  );
}

export default HomePage;


### frontend/src/pages/RecommendationsPage.js - Recommendations Page

javascript
import React, { useState, useEffect } from 'react';
import { useSearchParams } from 'react-router-dom';
import '../styles/RecommendationsPage.css';

function RecommendationsPage() {
  const [searchParams] = useSearchParams();
  const [videos, setVideos] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  const mood = searchParams.get('mood') || 'happy';
  const interests = searchParams.get('interests') || 'comedy';
  const time = searchParams.get('time') || '20';

  useEffect(() => {
    const fetchVideos = async () => {
      try {
        setLoading(true);
        const response = await fetch(`/api/recommendations?mood=${mood}&interests=${interests}&time=${time}`);
        
        if (!response.ok) {
          throw new Error('Failed to fetch recommendations');
        }
        
        const data = await response.json();
        setVideos(data);
        setLoading(false);
      } catch (err) {
        setError(err.message);
        setLoading(false);
      }
    };

    fetchVideos();
  }, [mood, interests, time]);

  const handleWatchVideo = (videoId) => {
    window.open(`https://www.youtube.com/watch?v=${videoId}`, '_blank');
  };

  if (loading) {
    return <div className="loading">Loading recommendations...</div>;
  }

  if (error) {
    return <div className="error">Error: {error}</div>;
  }

  return (
    <div className="recommendations-container">
      <h1>Your Personalized Recommendations</h1>
      <div className="preferences-summary">
        <p>Based on: <span className="highlight">{mood}</span> mood, <span className="highlight">{interests}</span> interest, <span className="highlight">{time} minutes</span> available</p>
      </div>
      
      {videos.length === 0 ? (
        <div className="no-results">
          <p>No videos found matching your preferences. Try different options!</p>
        </div>
      ) : (
        <div className="video-grid">
          {videos.map((video) => (
            <div key={video.videoId} className="video-card">
              <div className="thumbnail">
                <img src={video.thumbnail} alt={video.title} />
                <span className="duration">{video.duration}</span>
              </div>
              <div className="video-info">
                <h3>{video.title}</h3>
                <p className="channel">{video.channelTitle}</p>
              </div>
              <button 
                className="watch-btn" 
                onClick={() => handleWatchVideo(video.videoId)}
              >
                Watch Now
              </button>
            </div>
          ))}
        </div>
      )}
    </div>
  );
}

export default RecommendationsPage;


### frontend/src/styles/App.css - Main App Styles

css
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background-color: #f7f9fc;
  color: #333;
  line-height: 1.6;
}

.app {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}

.main-content {
  flex: 1;
  width: 100%;
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

a {
  color: #4285f4;
  text-decoration: none;
}

a:hover {
  text-decoration: underline;
}

button {
  cursor: pointer;
}

.loading, .error {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 300px;
  font-size: 1.2rem;
}

.error {
  color: #d93025;
}


### frontend/src/styles/HomePage.css - Home Page Styles

css
.home-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 40px 0;
}

.hero {
  text-align: center;
  margin-bottom: 40px;
}

.hero h1 {
  font-size: 2.5rem;
  margin-bottom: 16px;
  color: #1a73e8;
}

.hero p {
  font-size: 1.2rem;
  color: #5f6368;
  max-width: 600px;
}

.preferences-form {
  background-color: white;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  padding: 30px;
  width: 100%;
  max-width: 500px;
}

.form-group {
  margin-bottom: 24px;
}

.form-group label {
  display: block;
  margin-bottom: 8px;
  font-weight: 500;
  color: #202124;
}

.form-group select {
  width: 100%;
  padding: 12px;
  border: 1px solid #dadce0;
  border-radius: 4px;
  font-size: 16px;
  color: #3c4043;
  background-color: white;
  appearance: none;
  background-image: url("data:image/svg+xml;charset=US-ASCII,%3Csvg xmlns='http://www.w3.org/2000/svg' width='14' height='14' viewBox='0 0 20 20'%3E%3Cpath fill='%235F6368' d='M7 10l5 5 5-5z'/%3E%3C/svg%3E");
  background-repeat: no-repeat;
  background-position: right 12px center;
}

.form-group select:focus {
  outline: none;
  border-color: #1a73e8;
}

.submit-btn {
  display: block;
  width: 100%;
  padding: 14px;
  background-color: #1a73e8;
  color: white;
  border: none;
  border-radius: 4px;
  font-size: 16px;
  font-weight: 500;
  cursor: pointer;
  transition: background-color 0.2s;
}

.submit-btn:hover {
  background-color: #1765cc;
}


### frontend/src/styles/RecommendationsPage.css - Recommendations Page Styles

css
.recommendations-container {
  padding: 20px 0;
}

.recommendations-container h1 {
  text-align: center;
  margin-bottom: 20px;
  color: #1a73e8;
}

.preferences-summary {
  text-align: center;
  margin-bottom: 30px;
  color: #5f6368;
}

.highlight {
  color: #1a73e8;
  font-weight: 500;
}

.video-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 24px;
}

.video-card {
  background-color: white;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  transition: transform 0.2s, box-shadow 0.2s;
}

.video-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.15);
}

.thumbnail {
  position: relative;
  width: 100%;
  height: 0;
  padding-bottom: 56.25%; /* 16:9 aspect ratio */
  overflow: hidden;
}

.thumbnail img {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.duration {
  position: absolute;
  bottom: 8px;
  right: 8px;
  background-color: rgba(0, 0, 0, 0.7);
  color: white;
  padding: 4px 6px;
  border-radius: 4px;
  font-size: 12px;
}

.video-info {
  padding: 16px;
}

.video-info h3 {
  font-size: 16px;
  margin-bottom: 8px;
  font-weight: 500;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.channel {
  font-size: 14px;
  color: #5f6368;
}

.watch-btn {
  display: block;
  width: 100%;
  padding: 12px;
  background-color: #1a73e8;
  color: white;
  border: none;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: background-color 0.2s;
}

.watch-btn:hover {
  background-color: #1765cc;
}

.no-results {
  text-align: center;
  padding: 40px 0;
  color: #5f6368;
}


### frontend/src/styles/Header.css - Header Styles

css
.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 16px 24px;
  background-color: white;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.logo a {
  font-size: 22px;
  font-weight: 700;
  color: #1a73e8;
  text-decoration: none;
}

.nav ul {
  display: flex;
  list-style: none;
}

.nav li {
  margin-left: 24px;
}

.nav a {
  color: #5f6368;
  font-weight: 500;
  text-decoration: none;
  transition: color 0.2s;
}

.nav a:hover {
  color: #1a73e8;
}


### frontend/src/styles/Footer.css - Footer Styles

css
.footer {
  background-color: #f2f2f2;
  padding: 20px;
  text-align: center;
  color: #5f6368;
  margin-top: 40px;
}


## Backend Code (Node.js/Express)

### backend/server.js - Entry Point

javascript
const app = require('./app');
const config = require('./config');

const PORT = process.env.PORT || 5000;

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
  console.log(`Environment: ${process.env.NODE_ENV || 'development'}`);
});


### backend/app.js - Express App Setup

javascript
const express = require('express');
const cors = require('cors');
const path = require('path');
const mongoose = require('mongoose');
const config = require('./config');
const recommendationsRoutes = require('./routes/recommendations');

// Initialize Express app
const app = express();

// Middleware
app.use(cors());
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// Connect to MongoDB (if using database)
if (config.MONGODB_URI) {
  mongoose.connect(config.MONGODB_URI)
    .then(() => console.log('Connected to MongoDB'))
    .catch(err => console.error('MongoDB connection error:', err));
}

// API Routes
app.use('/api/recommendations', recommendationsRoutes);

// Serve static assets in production
if (process.env.NODE_ENV === 'production') {
  // Set static folder
  app.use(express.static(path.join(__dirname, '../frontend/build')));

  app.get('*', (req, res) => {
    res.sendFile(path.resolve(__dirname, '../frontend', 'build', 'index.html'));
  });
}

// Error handling middleware
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ message: 'Something went wrong on the server' });
});

module.exports = app;


### backend/config/index.js - Configuration

javascript
require('dotenv').config();

module.exports = {
  NODE_ENV: process.env.NODE_ENV || 'development',
  YOUTUBE_API_KEY: process.env.YOUTUBE_API_KEY,
  MONGODB_URI: process.env.MONGODB_URI,
  PORT: process.env.PORT || 5000
};


### backend/routes/recommendations.js - API Routes

javascript
const express = require('express');
const router = express.Router();
const recommendationsController = require('../controllers/recommendationsController');

// GET /api/recommendations - Get video recommendations
router.get('/', recommendationsController.getRecommendations);

module.exports = router;


### backend/controllers/recommendationsController.js - Controllers

javascript
const youtubeService = require('../services/youtubeService');

// Get video recommendations based on mood, interests, and time
exports.getRecommendations = async (req, res) => {
  try {
    const { mood, interests, time } = req.query;
    
    // Validate query parameters
    if (!mood || !interests || !time) {
      return res.status(400).json({ message: 'Missing required parameters' });
    }
    
    // Get video duration based on time parameter
    const duration = getDuration(time);
    
    // Create search query based on mood and interests
    const query = `${mood} ${interests}`;
    
    // Fetch videos from YouTube API
    const videos = await youtubeService.fetchVideos(query, duration);
    
    res.status(200).json(videos);
  } catch (error) {
    console.error('Error getting recommendations:', error);
    res.status(500).json({ message: 'Failed to get recommendations', error: error.message });
  }
};

// Helper function to map time to YouTube API duration parameter
const getDuration = (time) => {
  // YouTube API duration parameters: any, short (< 4m), medium (4-20m), long (> 20m)
  // For our purposes, we'll always use 'medium' or 'long' based on selected time
  return time === '20' ? 'medium' : 'long';
};


### backend/services/youtubeService.js - YouTube API Service

javascript
const axios = require('axios');
const config = require('../config');

// Fetch videos from YouTube API
exports.fetchVideos = async (query, duration) => {
  try {
    // Check if API key exists
    if (!config.YOUTUBE_API_KEY) {
      throw new Error('YouTube API key is not configured');
    }
    
    // Construct search URL with query parameters
    const searchUrl = `https://www.googleapis.com/youtube/v3/search`;
    const searchParams = {
      part: 'snippet',
      maxResults: 10,
      q: query,
      type: 'video',
      videoDuration: duration,
      key: config.YOUTUBE_API_KEY
    };
    
    // Make search request
    const searchResponse = await axios.get(searchUrl, { params: searchParams });
    const videoIds = searchResponse.data.items.map(item => item.id.videoId).join(',');
    
    // If no videos found, return empty array
    if (!videoIds) {
      return [];
    }
    
    // Get additional video details including duration
    const videosUrl = `https://www.googleapis.com/youtube/v3/videos`;
    const videosParams = {
      part: 'contentDetails,snippet,statistics',
      id: videoIds,
      key: config.YOUTUBE_API_KEY
    };
    
    const videosResponse = await axios.get(videosUrl, { params: videosParams });
    
    // Parse and format videos data
    const videos = videosResponse.data.items.map(item => {
      // Parse ISO 8601 duration format
      const duration = formatDuration(item.contentDetails.duration);
      
      // Filter videos based on actual duration (between 15-35 minutes)
      const durationMinutes = parseISO8601Duration(item.contentDetails.duration);
      
      // Only include videos that are between 15-35 minutes
      if (durationMinutes < 15 || durationMinutes > 35) {
        return null;
      }
      
      return {
        videoId: item.id,
        title: item.snippet.title,
        description: item.snippet.description,
        thumbnail: item.snippet.thumbnails.high.url,
        channelTitle: item.snippet.channelTitle,
        duration: duration,
        viewCount: item.statistics.viewCount
      };
    }).filter(Boolean); // Remove null entries
    
    return videos;
  } catch (error) {
    console.error('YouTube API error:', error);
    throw new Error('Failed to fetch videos from YouTube API');
  }
};

// Helper function to format duration from ISO 8601 (PT1H20M30S) to readable format (1:20:30)
const formatDuration = (isoDuration) => {
  const match = isoDuration.match(/PT(?:(\d+)H)?(?:(\d+)M)?(?:(\d+)S)?/);
  
  if (!match) {
    return '0:00';
  }
  
  const hours = parseInt(match[1] || 0);
  const minutes = parseInt(match[2] || 0);
  const seconds = parseInt(match[3] || 0);
  
  if (hours > 0) {
    return `${hours}:${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`;
  } else {
    return `${minutes}:${seconds.toString().padStart(2, '0')}`;
  }
};

// Helper function to parse ISO 8601 duration to minutes
const parseISO8601Duration = (isoDuration) => {
  const match = isoDuration.match(/PT(?:(\d+)H)?(?:(\d+)M)?(?:(\d+)S)?/);
  
  if (!match) {
    return 0;
  }
  
  const hours = parseInt(match[1] || 0);
  const minutes = parseInt(match[2] || 0);
  const seconds = parseInt(match[3] || 0);
  
  return hours * 60 + minutes + seconds / 60;
};


### backend/models/User.js - Optional User Model for Future Expansion

javascript
const mongoose = require('mongoose');

const UserSchema = new mongoose.Schema({
  preferences: {
    mood: {
      type: String,
      enum: ['happy', 'relaxed', 'curious', 'energetic', 'focused'],
      default: 'happy'
    },
    interests: {
      type: String,
      default: 'comedy'
    },
    timeAvailable: {
      type: String,
      enum: ['20', '25', '30'],
      default: '20'
    }
  },
  watchHistory: [{
    videoId: {
      type: String,
      required: true
    },
    title: String,
    watchedAt: {
      type: Date,
      default: Date.now
    }
  }],
  createdAt: {
    type: Date,
    default: Date.now
  }
});

module.exports = mongoose.model('User', UserSchema);


## Package Files

### frontend/package.json

json
{
  "name": "video-recommender-frontend",
  "version": "0.1.0",
  "private": true,
  "dependencies": {
    "@testing-library/jest-dom": "^5.16.5",
    "@testing-library/react": "^13.4.0",
    "@testing-library/user-event": "^13.5.0",
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-router-dom": "^6.11.1",
    "react-scripts": "5.0.1",
    "web-vitals": "^2.1.4"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject"
  },
  "eslintConfig": {
    "extends": [
      "react-app",
      "react-app/jest"
    ]
  },
  "browserslist": {
    "production": [
      ">0.2%",
      "not dead",
      "not op_mini all"
    ],
    "development": [
      "last 1 chrome version",
      "last 1 firefox version",
      "last 1 safari version"
    ]
  },
  "proxy": "http://localhost:5000"
}


### backend/package.json

json
{
  "name": "video-recommender-backend",
  "version": "1.0.0",
  "description": "Backend for video recommendation app",
  "main": "server.js",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  },
  "dependencies": {
    "axios": "^1.4.0",
    "cors": "^2.8.5",
    "dotenv": "^16.0.3",
    "express": "^4.18.2",
    "mongoose": "^7.1.0"
  },
  "devDependencies": {
    "nodemon": "^2.0.22"
  }
}


## Environment Files

### .env (Create this in the root directory)


# API Keys
YOUTUBE_API_KEY=YOUR_YOUTUBE_API_KEY_HERE

# Database
MONGODB_URI=mongodb+srv://username:password@cluster.mongodb.net/video-recommender

# Server Configuration
PORT=5000
NODE_ENV=development


## Project Setup Instructions

1. *Clone or create the project structure as shown above*

2. *Install dependencies*
   bash
   # Install backend dependencies
   cd backend
   npm install

   # Install frontend dependencies
   cd ../frontend
   npm install
   

3. **Create a .env file in the root directory**
   - Add your YouTube API key (Get one from Google Cloud Console)
   - Add MongoDB URI if you want to use database features

4. *Start the development servers*
   bash
   # Start backend server
   cd backend
   npm run dev

   # In a separate terminal, start frontend server
   cd frontend
   npm start
   

5. *Access the application*
   - Frontend: http://localhost:3000
   - Backend API: http://localhost:5000

## Next Steps for Further Development

1. Implement user authentication for saved preferences
2. Add more content sources beyond YouTube
3. Implement machine learning for better recommendations
4. Add social sharing features
5. Create mobile app version using React Native
