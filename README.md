## **Project Documentation: Drug Trafficking Detection System using MERN Stack**

### **Project Overview**

**Objective**: To develop a software solution that identifies users behind Telegram, WhatsApp, and Instagram-based drug trafficking. The solution focuses on detecting suspicious activities, identifying users through metadata, and using machine learning for anomaly detection.

### **Tech Stack**

- **Frontend**: React.js (for building the user interface)
- **Backend**: Node.js, Express.js (for server-side logic and API handling)
- **Database**: MongoDB (for storing logs, user details, and suspicious activities)
- **Real-Time Communication**: Socket.io (for notifications and alerts)
- **APIs**: Telegram Bot API, Instagram Graph API
- **Machine Learning**: Python (using Flask or FastAPI), Scikit-learn (for anomaly detection)
- **Device Fingerprinting**: FingerprintJS
- **IP Logging**: Custom middleware in Node.js

### **Project Modules**

1. **Data Collection and Monitoring**
2. **User Identification**
3. **Anomaly Detection with Machine Learning**
4. **Real-Time Alerts and Notifications**
5. **Reporting Dashboard**
6. **Collaboration with Authorities**

### **1. Data Collection and Monitoring**

**Objective**: Monitor suspicious activities on Telegram channels, WhatsApp groups, and Instagram handles for drug trafficking.

#### **1.1. Telegram Monitoring**

- **Create a Telegram Bot**: Use the Telegram Bot API to monitor specific channels and keywords.
  
  **Tools/Packages**: `node-telegram-bot-api`

  ```bash
  npm install node-telegram-bot-api
  ```

- **Implementation**: Create a bot that listens for messages in specific channels and groups. If the bot detects suspicious keywords (e.g., "MDMA", "LSD", "Mephedrone"), it logs the message details.

  ```javascript
  const TelegramBot = require('node-telegram-bot-api');
  const mongoose = require('mongoose');
  
  // Define the token for the Telegram bot
  const token = 'YOUR_TELEGRAM_BOT_API_TOKEN';
  const bot = new TelegramBot(token, { polling: true });
  
  // Define the Mongoose schema for suspicious messages
  const SuspiciousMessageSchema = new mongoose.Schema({
      platform: String,
      message: String,
      timestamp: Date,
      userDetails: Object,
  });
  const SuspiciousMessage = mongoose.model('SuspiciousMessage', SuspiciousMessageSchema);

  // Listen for messages
  bot.on('message', (msg) => {
      const chatId = msg.chat.id;
      const suspiciousKeywords = ['MDMA', 'LSD', 'Mephedrone'];
      
      if (suspiciousKeywords.some(keyword => msg.text.includes(keyword))) {
          // Log the message details into the database
          const message = new SuspiciousMessage({
              platform: 'Telegram',
              message: msg.text,
              timestamp: new Date(),
              userDetails: { username: msg.from.username, chatId: msg.chat.id }
          });
          message.save().then(() => console.log('Suspicious message saved!'));
      }
  });
  ```

#### **1.2. Instagram Monitoring**

- **Use Instagram Graph API**: Monitor specific hashtags and account activities for drug-related posts.

  **Tools/Packages**: `axios`

  ```bash
  npm install axios
  ```

- **Implementation**: Set up a service to fetch posts using specific hashtags. If a post contains suspicious content, log it.

  ```javascript
  const axios = require('axios');
  
  const accessToken = 'YOUR_INSTAGRAM_ACCESS_TOKEN';
  const tag = 'drugtrading'; // Example tag
  
  axios.get(`https://graph.instagram.com/v12.0/tags/${tag}/media?access_token=${accessToken}`)
      .then(response => {
          response.data.data.forEach(post => {
              // Log suspicious post details
              const suspiciousPost = new SuspiciousMessage({
                  platform: 'Instagram',
                  message: post.caption,
                  timestamp: new Date(),
                  userDetails: { userId: post.user_id, mediaId: post.id }
              });
              suspiciousPost.save().then(() => console.log('Suspicious post saved!'));
          });
      })
      .catch(error => console.error(error));
  ```

#### **1.3. WhatsApp Monitoring**

- **Limitations**: Direct monitoring is challenging due to encryption. Focus on analyzing metadata and user behavior.

### **2. User Identification**

**Objective**: Identify users behind suspicious activities using metadata like IP addresses, device fingerprints, etc.

#### **2.1. Device Fingerprinting**

- **Use FingerprintJS**: Integrate device fingerprinting to collect user information.

  **Tools/Packages**: `@fingerprintjs/fingerprintjs`

  ```bash
  npm install @fingerprintjs/fingerprintjs
  ```

- **Implementation**: Include a fingerprinting script in the React frontend to gather device information and send it to the backend.

  ```javascript
  import FingerprintJS from '@fingerprintjs/fingerprintjs';

  // Load the FingerprintJS agent
  const fpPromise = FingerprintJS.load();

  fpPromise
      .then(fp => fp.get())
      .then(result => {
          const visitorId = result.visitorId;
          console.log(visitorId); // Send this data to the backend
      });
  ```

#### **2.2. IP Address Logging**

- **Custom Middleware**: Implement a middleware in Express.js to capture IP addresses of incoming requests.

  ```javascript
  const express = require('express');
  const app = express();

  // Middleware to log IP addresses
  app.use((req, res, next) => {
      const ip = req.headers['x-forwarded-for'] || req.connection.remoteAddress;
      console.log(`IP address: ${ip}`);
      next();
  });

  app.listen(3000, () => console.log('Server running on port 3000'));
  ```

### **3. Anomaly Detection with Machine Learning**

**Objective**: Use ML models to detect anomalies in communication patterns indicating drug trafficking.

#### **3.1. Model Training and Deployment**

- **Python Flask API**: Train a model using Scikit-learn and expose it as an API.

  **Tools/Packages**: Scikit-learn, Flask

  ```python
  from sklearn.ensemble import IsolationForest
  import numpy as np
  from flask import Flask, request, jsonify

  app = Flask(__name__)

  # Train an Isolation Forest model
  model = IsolationForest(contamination=0.1)
  # Train your model with sample data (e.g., feature vectors representing user behavior)
  # model.fit(sample_data)

  @app.route('/predict', methods=['POST'])
  def predict():
      data = request.json['data']
      prediction = model.predict([data])
      return jsonify({'anomaly': prediction[0] == -1})

  if __name__ == '__main__':
      app.run(debug=True)
  ```

- **Integration with Node.js**: Call the Python API from the Node.js backend to check for anomalies.

  ```javascript
  const axios = require('axios');

  // Example function to call ML API
  function checkAnomaly(data) {
      axios.post('http://localhost:5000/predict', { data: [10, 2] })
          .then(response => {
              if (response.data.anomaly) {
                  console.log('Anomaly detected!');
              }
          })
          .catch(error => console.error(error));
  }
  ```

### **4. Real-Time Alerts and Notifications**

**Objective**: Notify administrators in real-time when suspicious activities are detected.

#### **4.1. Setting Up Socket.io for Real-Time Communication**

- **Backend Implementation**: Use Socket.io to emit real-time alerts when suspicious activity is detected.

  **Tools/Packages**: `socket.io`

  ```bash
  npm install socket.io
  ```

  ```javascript
  const http = require('http');
  const socketIo = require('socket.io');

  const server = http.createServer(app);
  const io = socketIo(server);

  // Emit an alert when a suspicious message is detected
  function sendAlert(message) {
      io.emit('alert', { message: message });
  }

  // Call sendAlert() whenever you detect suspicious activity
  ```

- **Frontend Implementation**: React component to listen for real-time alerts.

  ```javascript
  import io from 'socket.io-client';
  import React, { useEffect } from 'react';

  const socket = io('http://localhost:3000');

  const Alerts = () => {
      useEffect(() => {
          socket.on('alert', (data) => {
              alert(data.message); // Display the alert
          });
      }, []);

      return (
          <div>
              <h2>Alerts</h2>
          </div>
      );
  };

  export default Alerts;
  ```

### **5. Reporting Dashboard**

**Objective**: Visualize data on suspicious activities and provide analytics.

#### **5.1. Building a Dashboard with React**

- **Use Chart.js**: Integrate charting libraries to visualize trends and patterns.

  **Tools/Packages**: `chart.js`

  ```bash
  npm install chart.js react-chartjs-2
  ```

- **Implementation**: Create a React component to display charts.

  ```javascript
  import React,

 { useEffect, useState } from 'react';
  import { Line } from 'react-chartjs-2';
  import axios from 'axios';

  const Dashboard = () => {
      const [data, setData] = useState([]);

      useEffect(() => {
          axios.get('/api/suspicious-activities')
              .then(response => setData(response.data))
              .catch(error => console.error(error));
      }, []);

      return (
          <div>
              <h2>Suspicious Activity Dashboard</h2>
              <Line data={{
                  labels: data.map(item => item.date),
                  datasets: [
                      {
                          label: 'Number of Suspicious Activities',
                          data: data.map(item => item.count),
                          fill: false,
                          backgroundColor: 'red',
                          borderColor: 'red'
                      }
                  ]
              }} />
          </div>
      );
  };

  export default Dashboard;
  ```

### **6. Collaboration with Authorities**

**Objective**: Share identified suspicious data securely with law enforcement agencies.

#### **6.1. Data Sharing Mechanism**

- **Implementation**: Implement a secure API endpoint that authorized personnel can use to access suspicious activity logs. This endpoint should include authentication and authorization checks.

  ```javascript
  const express = require('express');
  const app = express();

  // Middleware to check for valid API keys
  function authenticate(req, res, next) {
      const apiKey = req.headers['api-key'];
      if (apiKey === 'YOUR_SECURE_API_KEY') {
          next();
      } else {
          res.status(401).send('Unauthorized');
      }
  }

  // Secure endpoint to get suspicious activities
  app.get('/api/suspicious-activities', authenticate, (req, res) => {
      SuspiciousMessage.find()
          .then(messages => res.json(messages))
          .catch(error => res.status(500).send('Error retrieving data'));
  });

  app.listen(3000, () => console.log('Server running on port 3000'));
  ```

### **Deployment**

- **Backend**: Deploy the Node.js and Python ML services using cloud platforms like AWS or Azure.
- **Frontend**: Deploy the React application on platforms like Vercel or Netlify.
- **Database**: Use a managed MongoDB service like MongoDB Atlas.

### **Testing and Validation**

- **Simulate Scenarios**: Use mock data to simulate different scenarios and verify system response.
- **Unit Tests**: Write unit tests for each module to ensure individual functionality.
- **Integration Tests**: Test the integration between different modules (e.g., Telegram monitoring with anomaly detection).

### **Future Enhancements**

- **WhatsApp Monitoring**: Explore legal and technical possibilities for more effective monitoring.
- **Advanced ML Models**: Use deep learning techniques for better anomaly detection.
- **Scalability**: Optimize the architecture for handling high traffic and large datasets.

### **Conclusion**

This documentation provides a comprehensive roadmap to implement the Drug Trafficking Detection System using the MERN stack and related technologies. By following these steps, your team can collaboratively develop a robust and scalable solution to detect and report drug trafficking activities on encrypted social media platforms.

---
