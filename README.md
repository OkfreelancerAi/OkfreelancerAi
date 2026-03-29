
databases:
  - name: analytics-dashboard-db
    databaseName: analytics_dashboard
    user: analytics_user
    plan: free

services:
  # Backend API (Node.js/Express)
  - type: web
    name: analytics-dashboard-api
    runtime: node
    plan: free
    rootDir: backend
    buildCommand: npm install
    startCommand: node server.js
    envVars:
      - key: NODE_ENV
        value: production
      - key: DATABASE_URL
        fromDatabase:
          name: analytics-dashboard-db
          property: connectionString
      - key: JWT_SECRET
        generateValue: true
      - key: PORT
        value: 3001
      - key: FRONTEND_URL
        value: https://analytics-dashboard-frontend.onrender.com

  # Frontend (React)
  - type: web
    name: analytics-dashboard-frontend
    runtime: static
    plan: free
    rootDir: frontend
    buildCommand: npm install && npm run build
    staticPublishPath: ./build
    envVars:
      - key: REACT_APP_API_URL
        value: https://analytics-dashboard-api.onrender.com/api
    routes:
      - type: rewrite
        source: /*
        destination: /index.html
