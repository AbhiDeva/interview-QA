# 25 Real-Time Data Coding Examples

## 1. Live Search with Debouncing

**Scenario:** Search products as user types with API call optimization

```javascript
class LiveSearch {
  constructor(apiEndpoint, delay = 300) {
    this.apiEndpoint = apiEndpoint;
    this.delay = delay;
    this.timeoutId = null;
  }

  debounce(func) {
    return (...args) => {
      clearTimeout(this.timeoutId);
      this.timeoutId = setTimeout(() => func.apply(this, args), this.delay);
    };
  }

  async search(query) {
    if (!query.trim()) {
      return [];
    }

    try {
      const response = await fetch(`${this.apiEndpoint}?q=${encodeURIComponent(query)}`);
      const data = await response.json();
      return data;
    } catch (error) {
      console.error('Search error:', error);
      return [];
    }
  }

  init(inputElement, resultsElement) {
    const debouncedSearch = this.debounce(async (query) => {
      resultsElement.innerHTML = '<li>Searching...</li>';
      const results = await this.search(query);
      
      if (results.length === 0) {
        resultsElement.innerHTML = '<li>No results found</li>';
      } else {
        resultsElement.innerHTML = results
          .map(item => `<li>${item.name}</li>`)
          .join('');
      }
    });

    inputElement.addEventListener('input', (e) => {
      debouncedSearch(e.target.value);
    });
  }
}

// Usage
const search = new LiveSearch('https://api.example.com/search');
const input = document.getElementById('search-input');
const results = document.getElementById('search-results');
search.init(input, results);
```

---

## 2. Real-Time Stock Price Ticker

**Scenario:** Display live stock prices with WebSocket

```javascript
class StockTicker {
  constructor(wsUrl) {
    this.wsUrl = wsUrl;
    this.ws = null;
    this.subscribers = new Map();
    this.priceHistory = new Map();
  }

  connect() {
    this.ws = new WebSocket(this.wsUrl);

    this.ws.onopen = () => {
      console.log('WebSocket connected');
      this.subscribe(['AAPL', 'GOOGL', 'MSFT']);
    };

    this.ws.onmessage = (event) => {
      const data = JSON.parse(event.data);
      this.handlePriceUpdate(data);
    };

    this.ws.onerror = (error) => {
      console.error('WebSocket error:', error);
    };

    this.ws.onclose = () => {
      console.log('WebSocket closed, reconnecting...');
      setTimeout(() => this.connect(), 5000);
    };
  }

  subscribe(symbols) {
    if (this.ws?.readyState === WebSocket.OPEN) {
      this.ws.send(JSON.stringify({
        action: 'subscribe',
        symbols: symbols
      }));
    }
  }

  handlePriceUpdate(data) {
    const { symbol, price, change, timestamp } = data;
    
    // Store price history
    if (!this.priceHistory.has(symbol)) {
      this.priceHistory.set(symbol, []);
    }
    this.priceHistory.get(symbol).push({ price, timestamp });

    // Keep only last 50 prices
    const history = this.priceHistory.get(symbol);
    if (history.length > 50) {
      history.shift();
    }

    // Notify subscribers
    if (this.subscribers.has(symbol)) {
      this.subscribers.get(symbol).forEach(callback => {
        callback({ symbol, price, change, history });
      });
    }
  }

  onPriceUpdate(symbol, callback) {
    if (!this.subscribers.has(symbol)) {
      this.subscribers.set(symbol, []);
    }
    this.subscribers.get(symbol).push(callback);
  }

  disconnect() {
    if (this.ws) {
      this.ws.close();
    }
  }
}

// Usage
const ticker = new StockTicker('wss://stock-api.example.com/prices');
ticker.connect();

ticker.onPriceUpdate('AAPL', (data) => {
  const element = document.getElementById('aapl-price');
  element.textContent = `$${data.price.toFixed(2)}`;
  element.className = data.change >= 0 ? 'price-up' : 'price-down';
});
```

---

## 3. Live Chat Application

**Scenario:** Real-time messaging with typing indicators

```javascript
class ChatRoom {
  constructor(roomId, userId, userName) {
    this.roomId = roomId;
    this.userId = userId;
    this.userName = userName;
    this.messages = [];
    this.typingUsers = new Set();
    this.ws = null;
  }

  connect() {
    this.ws = new WebSocket(`wss://chat-server.com/room/${this.roomId}`);

    this.ws.onopen = () => {
      this.ws.send(JSON.stringify({
        type: 'join',
        userId: this.userId,
        userName: this.userName
      }));
    };

    this.ws.onmessage = (event) => {
      const data = JSON.parse(event.data);
      this.handleMessage(data);
    };
  }

  handleMessage(data) {
    switch (data.type) {
      case 'message':
        this.addMessage(data);
        break;
      case 'typing':
        this.handleTyping(data);
        break;
      case 'stop-typing':
        this.handleStopTyping(data);
        break;
      case 'user-joined':
        this.handleUserJoined(data);
        break;
      case 'user-left':
        this.handleUserLeft(data);
        break;
    }
  }

  sendMessage(text) {
    const message = {
      type: 'message',
      userId: this.userId,
      userName: this.userName,
      text: text,
      timestamp: new Date().toISOString()
    };

    this.ws.send(JSON.stringify(message));
    this.addMessage(message);
  }

  addMessage(message) {
    this.messages.push(message);
    this.renderMessage(message);
  }

  renderMessage(message) {
    const messageElement = document.createElement('div');
    messageElement.className = `message ${message.userId === this.userId ? 'own' : 'other'}`;
    messageElement.innerHTML = `
      <div class="message-header">
        <span class="user-name">${message.userName}</span>
        <span class="timestamp">${new Date(message.timestamp).toLocaleTimeString()}</span>
      </div>
      <div class="message-text">${this.escapeHtml(message.text)}</div>
    `;
    document.getElementById('messages').appendChild(messageElement);
    this.scrollToBottom();
  }

  startTyping() {
    this.ws.send(JSON.stringify({
      type: 'typing',
      userId: this.userId,
      userName: this.userName
    }));
  }

  stopTyping() {
    this.ws.send(JSON.stringify({
      type: 'stop-typing',
      userId: this.userId
    }));
  }

  handleTyping(data) {
    if (data.userId !== this.userId) {
      this.typingUsers.add(data.userName);
      this.updateTypingIndicator();
    }
  }

  handleStopTyping(data) {
    this.typingUsers.delete(data.userName);
    this.updateTypingIndicator();
  }

  updateTypingIndicator() {
    const indicator = document.getElementById('typing-indicator');
    if (this.typingUsers.size > 0) {
      const users = Array.from(this.typingUsers).join(', ');
      indicator.textContent = `${users} ${this.typingUsers.size === 1 ? 'is' : 'are'} typing...`;
      indicator.style.display = 'block';
    } else {
      indicator.style.display = 'none';
    }
  }

  escapeHtml(text) {
    const div = document.createElement('div');
    div.textContent = text;
    return div.innerHTML;
  }

  scrollToBottom() {
    const container = document.getElementById('messages');
    container.scrollTop = container.scrollHeight;
  }

  handleUserJoined(data) {
    console.log(`${data.userName} joined the chat`);
  }

  handleUserLeft(data) {
    console.log(`${data.userName} left the chat`);
  }
}

// Usage
const chat = new ChatRoom('room-123', 'user-456', 'John Doe');
chat.connect();

const input = document.getElementById('message-input');
let typingTimer;

input.addEventListener('input', () => {
  chat.startTyping();
  clearTimeout(typingTimer);
  typingTimer = setTimeout(() => chat.stopTyping(), 1000);
});

input.addEventListener('keypress', (e) => {
  if (e.key === 'Enter') {
    chat.sendMessage(input.value);
    input.value = '';
    chat.stopTyping();
  }
});
```

---

## 4. Live Dashboard with Auto-Refresh

**Scenario:** Dashboard that updates metrics every 5 seconds

```javascript
class LiveDashboard {
  constructor(apiEndpoint, refreshInterval = 5000) {
    this.apiEndpoint = apiEndpoint;
    this.refreshInterval = refreshInterval;
    this.intervalId = null;
    this.metrics = {};
    this.charts = {};
  }

  async fetchMetrics() {
    try {
      const response = await fetch(this.apiEndpoint);
      const data = await response.json();
      return data;
    } catch (error) {
      console.error('Failed to fetch metrics:', error);
      return null;
    }
  }

  async updateMetrics() {
    const newMetrics = await this.fetchMetrics();
    if (!newMetrics) return;

    // Calculate changes
    const changes = this.calculateChanges(this.metrics, newMetrics);
    this.metrics = newMetrics;

    // Update UI
    this.renderMetrics(changes);
  }

  calculateChanges(oldMetrics, newMetrics) {
    const changes = {};
    for (const key in newMetrics) {
      if (oldMetrics[key] !== undefined) {
        changes[key] = {
          value: newMetrics[key],
          change: newMetrics[key] - oldMetrics[key],
          percentChange: ((newMetrics[key] - oldMetrics[key]) / oldMetrics[key] * 100).toFixed(2)
        };
      } else {
        changes[key] = {
          value: newMetrics[key],
          change: 0,
          percentChange: 0
        };
      }
    }
    return changes;
  }

  renderMetrics(changes) {
    for (const [key, data] of Object.entries(changes)) {
      const element = document.getElementById(`metric-${key}`);
      if (!element) continue;

      // Update value with animation
      this.animateValue(element.querySelector('.value'), data.value);

      // Update change indicator
      const changeElement = element.querySelector('.change');
      if (changeElement) {
        changeElement.textContent = `${data.change >= 0 ? '+' : ''}${data.change} (${data.percentChange}%)`;
        changeElement.className = `change ${data.change >= 0 ? 'positive' : 'negative'}`;
      }

      // Add flash effect
      element.classList.add('updated');
      setTimeout(() => element.classList.remove('updated'), 500);
    }

    // Update timestamp
    document.getElementById('last-updated').textContent = 
      `Last updated: ${new Date().toLocaleTimeString()}`;
  }

  animateValue(element, endValue, duration = 500) {
    const startValue = parseFloat(element.textContent) || 0;
    const startTime = performance.now();

    const animate = (currentTime) => {
      const elapsed = currentTime - startTime;
      const progress = Math.min(elapsed / duration, 1);

      const currentValue = startValue + (endValue - startValue) * progress;
      element.textContent = Math.round(currentValue).toLocaleString();

      if (progress < 1) {
        requestAnimationFrame(animate);
      }
    };

    requestAnimationFrame(animate);
  }

  start() {
    // Initial fetch
    this.updateMetrics();

    // Set up auto-refresh
    this.intervalId = setInterval(() => {
      this.updateMetrics();
    }, this.refreshInterval);
  }

  stop() {
    if (this.intervalId) {
      clearInterval(this.intervalId);
      this.intervalId = null;
    }
  }

  setRefreshInterval(interval) {
    this.stop();
    this.refreshInterval = interval;
    this.start();
  }
}

// Usage
const dashboard = new LiveDashboard('https://api.example.com/metrics');
dashboard.start();

// Manual refresh button
document.getElementById('refresh-btn').addEventListener('click', () => {
  dashboard.updateMetrics();
});

// Change refresh interval
document.getElementById('interval-select').addEventListener('change', (e) => {
  dashboard.setRefreshInterval(parseInt(e.target.value));
});
```

---

## 5. Real-Time Notifications System

**Scenario:** Push notifications with toast messages

```javascript
class NotificationManager {
  constructor() {
    this.notifications = [];
    this.container = null;
    this.maxNotifications = 5;
    this.defaultDuration = 5000;
    this.eventSource = null;
  }

  init(containerId) {
    this.container = document.getElementById(containerId);
    if (!this.container) {
      this.container = document.createElement('div');
      this.container.id = containerId;
      this.container.className = 'notification-container';
      document.body.appendChild(this.container);
    }
  }

  connectToServer(url) {
    this.eventSource = new EventSource(url);

    this.eventSource.onmessage = (event) => {
      const notification = JSON.parse(event.data);
      this.show(notification);
    };

    this.eventSource.addEventListener('notification', (event) => {
      const notification = JSON.parse(event.data);
      this.show(notification);
    });

    this.eventSource.onerror = (error) => {
      console.error('EventSource error:', error);
      this.show({
        type: 'error',
        message: 'Connection to notification server lost',
        duration: 3000
      });
    };
  }

  show(options) {
    const {
      type = 'info',
      title = '',
      message,
      duration = this.defaultDuration,
      action = null
    } = options;

    // Limit max notifications
    if (this.notifications.length >= this.maxNotifications) {
      this.remove(this.notifications[0].id);
    }

    const id = Date.now() + Math.random();
    const notification = this.createNotification(id, type, title, message, action);

    this.notifications.push({ id, element: notification });
    this.container.appendChild(notification);

    // Animate in
    setTimeout(() => notification.classList.add('show'), 10);

    // Auto remove
    if (duration > 0) {
      setTimeout(() => this.remove(id), duration);
    }

    // Play sound based on type
    this.playSound(type);

    return id;
  }

  createNotification(id, type, title, message, action) {
    const notification = document.createElement('div');
    notification.className = `notification notification-${type}`;
    notification.dataset.id = id;

    const icon = this.getIcon(type);

    notification.innerHTML = `
      <div class="notification-icon">${icon}</div>
      <div class="notification-content">
        ${title ? `<div class="notification-title">${title}</div>` : ''}
        <div class="notification-message">${message}</div>
        ${action ? `<button class="notification-action">${action.label}</button>` : ''}
      </div>
      <button class="notification-close">&times;</button>
    `;

    // Close button handler
    notification.querySelector('.notification-close').addEventListener('click', () => {
      this.remove(id);
    });

    // Action button handler
    if (action) {
      notification.querySelector('.notification-action').addEventListener('click', () => {
        action.callback();
        this.remove(id);
      });
    }

    return notification;
  }

  remove(id) {
    const index = this.notifications.findIndex(n => n.id === id);
    if (index === -1) return;

    const notification = this.notifications[index].element;
    notification.classList.remove('show');

    setTimeout(() => {
      notification.remove();
      this.notifications.splice(index, 1);
    }, 300);
  }

  getIcon(type) {
    const icons = {
      success: '✓',
      error: '✕',
      warning: '⚠',
      info: 'ℹ'
    };
    return icons[type] || icons.info;
  }

  playSound(type) {
    // Optional: play notification sound
    const audio = new Audio(`/sounds/notification-${type}.mp3`);
    audio.volume = 0.3;
    audio.play().catch(() => {}); // Ignore errors
  }

  clear() {
    this.notifications.forEach(n => n.element.remove());
    this.notifications = [];
  }

  disconnect() {
    if (this.eventSource) {
      this.eventSource.close();
    }
  }
}

// Usage
const notifications = new NotificationManager();
notifications.init('notification-container');
notifications.connectToServer('/api/notifications/stream');

// Manual notification
notifications.show({
  type: 'success',
  title: 'Order Completed',
  message: 'Your order #12345 has been processed',
  duration: 5000,
  action: {
    label: 'View Order',
    callback: () => window.location.href = '/orders/12345'
  }
});
```

---

## 6. Live Collaboration - Cursor Tracking

**Scenario:** Show other users' cursors in real-time

```javascript
class CollaborativeCursors {
  constructor(documentId, userId, userName) {
    this.documentId = documentId;
    this.userId = userId;
    this.userName = userName;
    this.cursors = new Map();
    this.ws = null;
    this.throttleDelay = 50;
    this.lastSent = 0;
  }

  connect() {
    this.ws = new WebSocket(`wss://collab-server.com/doc/${this.documentId}`);

    this.ws.onopen = () => {
      this.ws.send(JSON.stringify({
        type: 'join',
        userId: this.userId,
        userName: this.userName
      }));
    };

    this.ws.onmessage = (event) => {
      const data = JSON.parse(event.data);
      this.handleMessage(data);
    };

    // Track local cursor
    document.addEventListener('mousemove', (e) => {
      this.sendCursorPosition(e.clientX, e.clientY);
    });

    document.addEventListener('mouseleave', () => {
      this.sendCursorHidden();
    });
  }

  sendCursorPosition(x, y) {
    const now = Date.now();
    if (now - this.lastSent < this.throttleDelay) return;

    this.lastSent = now;

    this.ws.send(JSON.stringify({
      type: 'cursor-move',
      userId: this.userId,
      userName: this.userName,
      x: x,
      y: y,
      timestamp: now
    }));
  }

  sendCursorHidden() {
    this.ws.send(JSON.stringify({
      type: 'cursor-hide',
      userId: this.userId
    }));
  }

  handleMessage(data) {
    switch (data.type) {
      case 'cursor-move':
        this.updateCursor(data);
        break;
      case 'cursor-hide':
        this.hideCursor(data.userId);
        break;
      case 'user-left':
        this.removeCursor(data.userId);
        break;
    }
  }

  updateCursor(data) {
    if (data.userId === this.userId) return;

    let cursor = this.cursors.get(data.userId);
    
    if (!cursor) {
      cursor = this.createCursor(data.userId, data.userName);
      this.cursors.set(data.userId, cursor);
      document.body.appendChild(cursor.element);
    }

    cursor.element.style.left = data.x + 'px';
    cursor.element.style.top = data.y + 'px';
    cursor.element.style.display = 'block';

    // Clear hide timeout
    if (cursor.hideTimeout) {
      clearTimeout(cursor.hideTimeout);
    }

    // Auto-hide after 3 seconds of inactivity
    cursor.hideTimeout = setTimeout(() => {
      cursor.element.style.display = 'none';
    }, 3000);
  }

  createCursor(userId, userName) {
    const element = document.createElement('div');
    element.className = 'collab-cursor';
    element.innerHTML = `
      <svg width="24" height="24" viewBox="0 0 24 24">
        <path d="M0 0 L0 20 L7 13 L12 24 L14 23 L9 12 L18 12 Z" 
              fill="${this.getUserColor(userId)}"/>
      </svg>
      <div class="cursor-label">${userName}</div>
    `;
    
    return { element, hideTimeout: null };
  }

  getUserColor(userId) {
    const colors = [
      '#FF6B6B', '#4ECDC4', '#45B7D1', '#FFA07A',
      '#98D8C8', '#F7DC6F', '#BB8FCE', '#85C1E2'
    ];
    const hash = userId.split('').reduce((acc, char) => 
      acc + char.charCodeAt(0), 0);
    return colors[hash % colors.length];
  }

  hideCursor(userId) {
    const cursor = this.cursors.get(userId);
    if (cursor) {
      cursor.element.style.display = 'none';
    }
  }

  removeCursor(userId) {
    const cursor = this.cursors.get(userId);
    if (cursor) {
      cursor.element.remove();
      this.cursors.delete(userId);
    }
  }

  disconnect() {
    if (this.ws) {
      this.ws.close();
    }
    this.cursors.forEach(cursor => cursor.element.remove());
    this.cursors.clear();
  }
}

// Usage
const cursors = new CollaborativeCursors('doc-123', 'user-456', 'John Doe');
cursors.connect();
```

---

## 7. Real-Time Form Validation

**Scenario:** Validate form fields as user types with async checks

```javascript
class RealTimeValidator {
  constructor(formId) {
    this.form = document.getElementById(formId);
    this.fields = new Map();
    this.validationResults = new Map();
    this.debounceTimers = new Map();
  }

  addField(fieldId, rules) {
    const field = document.getElementById(fieldId);
    if (!field) return;

    this.fields.set(fieldId, { element: field, rules });

    field.addEventListener('input', () => {
      this.validateField(fieldId);
    });

    field.addEventListener('blur', () => {
      this.validateField(fieldId, true);
    });
  }

  async validateField(fieldId, showErrors = false) {
    const field = this.fields.get(fieldId);
    if (!field) return;

    const value = field.element.value;
    const errors = [];

    // Clear previous debounce timer
    if (this.debounceTimers.has(fieldId)) {
      clearTimeout(this.debounceTimers.get(fieldId));
    }

    // Debounce validation
    return new Promise((resolve) => {
      const timer = setTimeout(async () => {
        // Run validation rules
        for (const rule of field.rules) {
          const result = await this.runRule(rule, value, field.element);
          if (!result.valid) {
            errors.push(result.message);
          }
        }

        this.validationResults.set(fieldId, {
          valid: errors.length === 0,
          errors
        });

        if (showErrors || errors.length > 0) {
          this.showFieldErrors(fieldId, errors);
        }

        resolve(errors.length === 0);
      }, 300);

      this.debounceTimers.set(fieldId, timer);
    });
  }

  async runRule(rule, value, element) {
    switch (rule.type) {
      case 'required':
        return {
          valid: value.trim().length > 0,
          message: rule.message || 'This field is required'
        };

      case 'email':
        const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
        return {
          valid: emailRegex.test(value),
          message: rule.message || 'Please enter a valid email'
        };

      case 'minLength':
        return {
          valid: value.length >= rule.value,
          message: rule.message || `Minimum ${rule.value} characters required`
        };

      case 'maxLength':
        return {
          valid: value.length <= rule.value,
          message: rule.message || `Maximum ${rule.value} characters allowed`
        };

      case 'pattern':
        return {
          valid: new RegExp(rule.pattern).test(value),
          message: rule.message || 'Invalid format'
        };

      case 'async':
        // For async validation (e.g., check username availability)
        try {
          const response = await fetch(rule.url, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ value })
          });
          const data = await response.json();
          return {
            valid: data.available,
            message: rule.message || data.message
          };
        } catch (error) {
          return {
            valid: false,
            message: 'Validation error occurred'
          };
        }

      case 'custom':
        return rule.validator(value, element);

      default:
        return { valid: true, message: '' };
    }
  }

  showFieldErrors(fieldId, errors) {
    const field = this.fields.get(fieldId);
    if (!field) return;

    const errorContainer = field.element.parentElement.querySelector('.field-errors')
      || this.createErrorContainer(field.element);

    if (errors.length > 0) {
      field.element.classList.add('invalid');
      field.element.classList.remove('valid');
      errorContainer.innerHTML = errors
        .map(err => `<div class="error-message">${err}</div>`)
        .join('');
      errorContainer.style.display = 'block';
    } else {
      field.element.classList.remove('invalid');
      field.element.classList.add('valid');
      errorContainer.style.display = 'none';
    }
  }

  createErrorContainer(fieldElement) {
    const container = document.createElement('div');
    container.className = 'field-errors';
    fieldElement.parentElement.appendChild(container);
    return container;
  }

  async validateAll() {
    const validations = Array.from(this.fields.keys()).map(fieldId =>
      this.validateField(fieldId, true)
    );
    const results = await Promise.all(validations);
    return results.every(result => result);
  }

  getErrors() {
    const errors = {};
    this.validationResults.forEach((result, fieldId) => {
      if (!result.valid) {
        errors[fieldId] = result.errors;
      }
    });
    return errors;
  }
}

// Usage
const validator = new RealTimeValidator('registration-form');

validator.addField('username', [
  { type: 'required' },
  { type: 'minLength', value: 3 },
  { type: 'pattern', pattern: '^[a-zA-Z0-9_]+$', message: 'Only letters, numbers, and underscores' },
  { 
    type: 'async', 
    url: '/api/check-username',
    message: 'Username is already taken'
  }
]);

validator.addField('email', [
  { type: 'required' },
  { type: 'email' },
  { 
    type: 'async', 
    url: '/api/check-email',
    message: 'Email is already registered'
  }
]);

validator.addField('password', [
  { type: 'required' },
  { type: 'minLength', value: 8 },
  {
    type: 'custom',
    validator: (value) => {
      const hasUpper = /[A-Z]/.test(value);
      const hasLower = /[a-z]/.test(value);
      const hasNumber = /[0-9]/.test(value);
      const valid = hasUpper && hasLower && hasNumber;
      return {
        valid,
        message: 'Password must contain uppercase, lowercase, and number'
      };
    }
  }
]);

// Handle form submission
document.getElementById('registration-form').addEventListener('submit', async (e) => {
  e.preventDefault();
  
  const isValid = await validator.validateAll();
  
  if (isValid) {
    console.log('Form is valid, submitting...');
    // Submit form
  } else {
    console.log('Form has errors:', validator.getErrors());
  }
});
```

---

## 8. Live Poll/Voting System

**Scenario:** Real-time voting with live results

```javascript
class LivePoll {
  constructor(pollId) {
    this.pollId = pollId;
    this.poll = null;
    this.hasVoted = false;
    this.ws = null;
  }

  async init() {
    await this.loadPoll();
    this.connectWebSocket();
    this.render();
  }

  async loadPoll() {
    try {
      const response = await fetch(`/api/polls/${this.pollId}`);
      this.poll = await response.json();
      this.hasVoted = this.poll.userVote !== null;
    } catch (error) {
      console.error('Failed to load poll:', error);
    }
  }

  connectWebSocket() {
    this.ws = new WebSocket(`wss://poll-server.com/poll/${this.pollId}`);

    this.ws.onmessage = (event) => {
      const data = JSON.parse(event.data);
      if (data.type === 'vote-update') {
        this.updateResults(data.results);
      }
    };
  }

  async vote(optionId) {
    if (this.hasVoted) return;

    try {
      const response = await fetch(`/api/polls/${this.pollId}/vote`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ optionId })
      });

      const result = await response.json();
      this.hasVoted = true;
      this.poll.userVote = optionId;
      this.updateResults(result.results);
    } catch (error) {
      console.error('Vote failed:', error);
    }
  }

  updateResults(results) {
    this.poll.options = this.poll.options.map(option => ({
      ...option,
      votes: results[option.id] || 0
    }));

    const totalVotes = Object.values(results).reduce((sum, votes) => sum + votes, 0);
    this.poll.totalVotes = totalVotes;

    this.render();
  }

  render() {
    const container = document.getElementById(`poll-${this.pollId}`);
    if (!container) return;

    container.innerHTML = `
      <div class="poll-container">
        <h3 class="poll-question">${this.poll.question}</h3>
        <div class="poll-options">
          ${this.poll.options.map(option => this.renderOption(option)).join('')}
        </div>
        <div class="poll-footer">
          <span class="total-votes">${this.poll.totalVotes} votes</span>
        </div>
      </div>
    `;

    if (!this.hasVoted) {
      container.querySelectorAll('.poll-option').forEach(el => {
        el.addEventListener('click', () => {
          this.vote(el.dataset.optionId);
        });
      });
    }
  }

  renderOption(option) {
    const percentage = this.poll.totalVotes > 0
      ? (option.votes / this.poll.totalVotes * 100).toFixed(1)
      : 0;

    const isSelected = this.hasVoted && this.poll.userVote === option.id;

    return `
      <div class="poll-option ${this.hasVoted ? 'voted' : 'clickable'} ${isSelected ? 'selected' : ''}"
           data-option-id="${option.id}">
        <div class="option-content">
          <span class="option-text">${option.text}</span>
          <span class="option-stats">
            ${this.hasVoted ? `${percentage}% (${option.votes})` : ''}
          </span>
        </div>
        ${this.hasVoted ? `
          <div class="option-bar">
            <div class="option-bar-fill" style="width: ${percentage}%"></div>
          </div>
        ` : ''}
      </div>
    `;
  }

  disconnect() {
    if (this.ws) {
      this.ws.close();
    }
  }
}

// Usage
const poll = new LivePoll('poll-123');
poll.init();
```

---

## 9. Real-Time Location Tracking

**Scenario:** Track and display user location on a map

```javascript
class LocationTracker {
  constructor(userId, mapElementId) {
    this.userId = userId;
    this.mapElement = document.getElementById(mapElementId);
    this.map = null;
    this.markers = new Map();
    this.watchId = null;
    this.ws = null;
    this.updateInterval = 5000; // 5 seconds
  }

  async init() {
    await this.initMap();
    this.startTracking();
    this.connectWebSocket();
  }

  async initMap() {
    // Using Leaflet.js as example
    this.map = L.map(this.mapElement).setView([0, 0], 13);
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      attribution: '© OpenStreetMap contributors'
    }).addTo(this.map);
  }

  startTracking() {
    if (!navigator.geolocation) {
      console.error('Geolocation not supported');
      return;
    }

    this.watchId = navigator.geolocation.watchPosition(
      (position) => {
        const { latitude, longitude, accuracy } = position.coords;
        this.updatePosition(latitude, longitude, accuracy);
      },
      (error) => {
        console.error('Geolocation error:', error);
      },
      {
        enableHighAccuracy: true,
        maximumAge: 0,
        timeout: 5000
      }
    );
  }

  updatePosition(lat, lng, accuracy) {
    // Update own marker
    this.updateMarker(this.userId, lat, lng, 'You', true);

    // Send to server
    if (this.ws?.readyState === WebSocket.OPEN) {
      this.ws.send(JSON.stringify({
        type: 'location-update',
        userId: this.userId,
        lat,
        lng,
        accuracy,
        timestamp: Date.now()
      }));
    }

    // Center map on own location
    this.map.setView([lat, lng], 13);
  }

  connectWebSocket() {
    this.ws = new WebSocket('wss://location-server.com/track');

    this.ws.onopen = () => {
      this.ws.send(JSON.stringify({
        type: 'join',
        userId: this.userId
      }));
    };

    this.ws.onmessage = (event) => {
      const data = JSON.parse(event.data);
      this.handleLocationUpdate(data);
    };
  }

  handleLocationUpdate(data) {
    if (data.userId !== this.userId) {
      this.updateMarker(
        data.userId,
        data.lat,
        data.lng,
        data.userName || `User ${data.userId}`,
        false
      );
    }
  }

  updateMarker(userId, lat, lng, label, isOwnMarker) {
    let marker = this.markers.get(userId);

    if (!marker) {
      const icon = L.divIcon({
        className: 'custom-marker',
        html: `
          <div class="marker ${isOwnMarker ? 'own-marker' : 'other-marker'}">
            <div class="marker-icon">📍</div>
            <div class="marker-label">${label}</div>
          </div>
        `
      });

      marker = L.marker([lat, lng], { icon }).addTo(this.map);
      this.markers.set(userId, marker);
    } else {
      marker.setLatLng([lat, lng]);
    }

    // Add animation
    marker.getElement()?.classList.add('marker-pulse');
    setTimeout(() => {
      marker.getElement()?.classList.remove('marker-pulse');
    }, 500);
  }

  stopTracking() {
    if (this.watchId) {
      navigator.geolocation.clearWatch(this.watchId);
    }
    if (this.ws) {
      this.ws.close();
    }
  }

  // Calculate distance between users
  calculateDistance(lat1, lng1, lat2, lng2) {
    const R = 6371; // Earth's radius in km
    const dLat = this.toRad(lat2 - lat1);
    const dLng = this.toRad(lng2 - lng1);
    const a =
      Math.sin(dLat / 2) * Math.sin(dLat / 2) +
      Math.cos(this.toRad(lat1)) * Math.cos(this.toRad(lat2)) *
      Math.sin(dLng / 2) * Math.sin(dLng / 2);
    const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
    return R * c;
  }

  toRad(degrees) {
    return degrees * (Math.PI / 180);
  }
}

// Usage
const tracker = new LocationTracker('user-123', 'map');
tracker.init();
```

---

## 10. Real-Time Activity Feed

**Scenario:** Social media style activity feed with live updates

```javascript
class ActivityFeed {
  constructor(feedId) {
    this.feedId = feedId;
    this.container = document.getElementById(feedId);
    this.activities = [];
    this.ws = null;
    this.page = 1;
    this.loading = false;
    this.hasMore = true;
  }

  init() {
    this.loadActivities();
    this.connectWebSocket();
    this.setupInfiniteScroll();
  }

  async loadActivities(page = 1) {
    if (this.loading) return;
    this.loading = true;

    try {
      const response = await fetch(`/api/activities?page=${page}&limit=20`);
      const data = await response.json();

      if (page === 1) {
        this.activities = data.activities;
        this.render();
      } else {
        this.activities.push(...data.activities);
        this.appendActivities(data.activities);
      }

      this.hasMore = data.hasMore;
    } catch (error) {
      console.error('Failed to load activities:', error);
    } finally {
      this.loading = false;
    }
  }

  connectWebSocket() {
    this.ws = new WebSocket('wss://activity-server.com/feed');

    this.ws.onmessage = (event) => {
      const activity = JSON.parse(event.data);
      this.addNewActivity(activity);
    };
  }

  addNewActivity(activity) {
    this.activities.unshift(activity);
    this.prependActivity(activity);
  }

  render() {
    this.container.innerHTML = `
      <div class="feed-header">
        <h2>Activity Feed</h2>
        <button class="refresh-btn">🔄</button>
      </div>
      <div class="activities">
        ${this.activities.map(a => this.renderActivity(a)).join('')}
      </div>
      ${this.hasMore ? '<div class="loading-more">Loading...</div>' : ''}
    `;

    this.container.querySelector('.refresh-btn').addEventListener('click', () => {
      this.loadActivities(1);
    });
  }

  renderActivity(activity) {
    const timeAgo = this.getTimeAgo(activity.timestamp);

    return `
      <div class="activity-item" data-id="${activity.id}">
        <div class="activity-avatar">
          <img src="${activity.user.avatar}" alt="${activity.user.name}">
        </div>
        <div class="activity-content">
          <div class="activity-header">
            <strong>${activity.user.name}</strong>
            <span class="activity-action">${this.getActionText(activity)}</span>
            <span class="activity-time">${timeAgo}</span>
          </div>
          <div class="activity-body">
            ${this.renderActivityBody(activity)}
          </div>
          <div class="activity-actions">
            <button class="like-btn" data-id="${activity.id}">
              ❤️ ${activity.likes || 0}
            </button>
            <button class="comment-btn" data-id="${activity.id}">
              💬 ${activity.comments || 0}
            </button>
            <button class="share-btn" data-id="${activity.id}">
              🔗 Share
            </button>
          </div>
        </div>
      </div>
    `;
  }

  getActionText(activity) {
    const actions = {
      post: 'created a post',
      comment: 'commented on',
      like: 'liked',
      share: 'shared',
      follow: 'started following'
    };
    return actions[activity.type] || 'performed an action';
  }

  renderActivityBody(activity) {
    switch (activity.type) {
      case 'post':
        return `<p>${activity.content}</p>`;
      case 'comment':
        return `<p>${activity.content}</p>`;
      case 'like':
        return `<p>Liked: "${activity.target.title}"</p>`;
      case 'share':
        return `<p>Shared: "${activity.target.title}"</p>`;
      default:
        return '';
    }
  }

  prependActivity(activity) {
    const activityHtml = this.renderActivity(activity);
    const temp = document.createElement('div');
    temp.innerHTML = activityHtml;
    const element = temp.firstElementChild;

    const container = this.container.querySelector('.activities');
    container.insertBefore(element, container.firstChild);

    // Animate in
    element.classList.add('activity-new');
    setTimeout(() => element.classList.remove('activity-new'), 500);
  }

  appendActivities(activities) {
    const container = this.container.querySelector('.activities');
    const fragment = document.createDocumentFragment();

    activities.forEach(activity => {
      const temp = document.createElement('div');
      temp.innerHTML = this.renderActivity(activity);
      fragment.appendChild(temp.firstElementChild);
    });

    container.appendChild(fragment);
  }

  setupInfiniteScroll() {
    const observer = new IntersectionObserver((entries) => {
      if (entries[0].isIntersecting && this.hasMore && !this.loading) {
        this.page++;
        this.loadActivities(this.page);
      }
    }, { threshold: 0.5 });

    const loadingElement = this.container.querySelector('.loading-more');
    if (loadingElement) {
      observer.observe(loadingElement);
    }
  }

  getTimeAgo(timestamp) {
    const seconds = Math.floor((Date.now() - new Date(timestamp)) / 1000);

    const intervals = {
      year: 31536000,
      month: 2592000,
      week: 604800,
      day: 86400,
      hour: 3600,
      minute: 60
    };

    for (const [unit, secondsInUnit] of Object.entries(intervals)) {
      const interval = Math.floor(seconds / secondsInUnit);
      if (interval >= 1) {
        return `${interval} ${unit}${interval > 1 ? 's' : ''} ago`;
      }
    }

    return 'Just now';
  }

  disconnect() {
    if (this.ws) {
      this.ws.close();
    }
  }
}

// Usage
const feed = new ActivityFeed('activity-feed');
feed.init();
```

---

## 11. Real-Time Progress Tracker

**Scenario:** Track file upload or processing progress

```javascript
class ProgressTracker {
  constructor(containerId) {
    this.container = document.getElementById(containerId);
    this.tasks = new Map();
  }

  createTask(taskId, filename, totalSize) {
    const task = {
      id: taskId,
      filename,
      totalSize,
      uploadedSize: 0,
      progress: 0,
      status: 'uploading',
      startTime: Date.now(),
      speed: 0,
      remainingTime: 0
    };

    this.tasks.set(taskId, task);
    this.renderTask(task);
    return task;
  }

  updateProgress(taskId, uploadedSize) {
    const task = this.tasks.get(taskId);
    if (!task) return;

    const elapsed = (Date.now() - task.startTime) / 1000; // seconds
    task.uploadedSize = uploadedSize;
    task.progress = (uploadedSize / task.totalSize) * 100;
    task.speed = uploadedSize / elapsed; // bytes per second
    task.remainingTime = (task.totalSize - uploadedSize) / task.speed;

    this.updateTaskUI(task);
  }

  completeTask(taskId) {
    const task = this.tasks.get(taskId);
    if (!task) return;

    task.status = 'completed';
    task.progress = 100;
    this.updateTaskUI(task);
  }

  failTask(taskId, error) {
    const task = this.tasks.get(taskId);
    if (!task) return;

    task.status = 'failed';
    task.error = error;
    this.updateTaskUI(task);
  }

  renderTask(task) {
    const taskElement = document.createElement('div');
    taskElement.className = 'progress-task';
    taskElement.id = `task-${task.id}`;
    taskElement.innerHTML = this.getTaskHTML(task);

    this.container.appendChild(taskElement);
  }

  updateTaskUI(task) {
    const element = document.getElementById(`task-${task.id}`);
    if (element) {
      element.innerHTML = this.getTaskHTML(task);
    }
  }

  getTaskHTML(task) {
    const statusIcons = {
      uploading: '⏳',
      completed: '✅',
      failed: '❌'
    };

    return `
      <div class="task-header">
        <span class="task-icon">${statusIcons[task.status]}</span>
        <span class="task-filename">${task.filename}</span>
        <span class="task-size">${this.formatSize(task.uploadedSize)} / ${this.formatSize(task.totalSize)}</span>
      </div>
      
      <div class="progress-bar">
        <div class="progress-fill" style="width: ${task.progress}%"></div>
      </div>
      
      <div class="task-stats">
        <span class="progress-percent">${task.progress.toFixed(1)}%</span>
        ${task.status === 'uploading' ? `
          <span class="upload-speed">${this.formatSpeed(task.speed)}</span>
          <span class="remaining-time">${this.formatTime(task.remainingTime)} remaining</span>
        ` : ''}
        ${task.status === 'failed' ? `
          <span class="error-message">${task.error}</span>
        ` : ''}
      </div>
    `;
  }

  formatSize(bytes) {
    const units = ['B', 'KB', 'MB', 'GB'];
    let size = bytes;
    let unitIndex = 0;

    while (size >= 1024 && unitIndex < units.length - 1) {
      size /= 1024;
      unitIndex++;
    }

    return `${size.toFixed(2)} ${units[unitIndex]}`;
  }

  formatSpeed(bytesPerSecond) {
    return `${this.formatSize(bytesPerSecond)}/s`;
  }

  formatTime(seconds) {
    if (!isFinite(seconds)) return '--:--';
    
    const mins = Math.floor(seconds / 60);
    const secs = Math.floor(seconds % 60);
    return `${mins}:${secs.toString().padStart(2, '0')}`;
  }

  removeTask(taskId) {
    const element = document.getElementById(`task-${taskId}`);
    if (element) {
      element.classList.add('fade-out');
      setTimeout(() => {
        element.remove();
        this.tasks.delete(taskId);
      }, 300);
    }
  }
}

// Usage with file upload
async function uploadFile(file) {
  const tracker = new ProgressTracker('upload-container');
  const taskId = `upload-${Date.now()}`;
  
  tracker.createTask(taskId, file.name, file.size);

  const formData = new FormData();
  formData.append('file', file);

  const xhr = new XMLHttpRequest();

  xhr.upload.addEventListener('progress', (e) => {
    if (e.lengthComputable) {
      tracker.updateProgress(taskId, e.loaded);
    }
  });

  xhr.addEventListener('load', () => {
    if (xhr.status === 200) {
      tracker.completeTask(taskId);
      setTimeout(() => tracker.removeTask(taskId), 3000);
    } else {
      tracker.failTask(taskId, 'Upload failed');
    }
  });

  xhr.addEventListener('error', () => {
    tracker.failTask(taskId, 'Network error');
  });

  xhr.open('POST', '/api/upload');
  xhr.send(formData);
}

// Handle file selection
document.getElementById('file-input').addEventListener('change', (e) => {
  Array.from(e.target.files).forEach(file => {
    uploadFile(file);
  });
});
```

---

## 12-25. Additional Real-Time Examples (Quick Reference)

### 12. Real-Time Collaborative Editor
```javascript
// Track document changes and sync with other users
class CollaborativeEditor {
  constructor(documentId) {
    this.documentId = documentId;
    this.editor = null;
    this.ws = null;
    this.pendingChanges = [];
  }

  trackChanges(change) {
    this.ws.send(JSON.stringify({
      type: 'change',
      documentId: this.documentId,
      change: change
    }));
  }
}
```

### 13. Live Sports Score Updates
```javascript
class LiveScoreboard {
  updateScore(gameId, homeScore, awayScore) {
    const element = document.getElementById(`game-${gameId}`);
    element.querySelector('.home-score').textContent = homeScore;
    element.querySelector('.away-score').textContent = awayScore;
  }
}
```

### 14. Real-Time Analytics Dashboard
```javascript
class AnalyticsDashboard {
  constructor() {
    this.metrics = {};
    this.charts = new Map();
  }

  updateMetric(name, value) {
    this.metrics[name] = value;
    this.animateCounter(name, value);
  }

  animateCounter(id, endValue) {
    // Animate number changes
  }
}
```

### 15. Live Auction Bidding
```javascript
class AuctionBidding {
  placeBid(itemId, amount) {
    this.ws.send(JSON.stringify({
      type: 'bid',
      itemId,
      amount,
      userId: this.userId
    }));
  }

  updateHighestBid(itemId, amount, bidderName) {
    // Update UI with new highest bid
  }
}
```

### 16. Real-Time Weather Updates
```javascript
class WeatherWidget {
  async fetchWeather(location) {
    const response = await fetch(`/api/weather/${location}`);
    const data = await response.json();
    this.updateUI(data);
  }

  scheduleUpdates() {
    setInterval(() => this.fetchWeather(this.location), 600000); // 10 min
  }
}
```

### 17. Live Translation Service
```javascript
class LiveTranslator {
  async translate(text, targetLang) {
    const response = await fetch('/api/translate', {
      method: 'POST',
      body: JSON.stringify({ text, targetLang })
    });
    return await response.json();
  }
}
```

### 18. Real-Time Inventory Management
```javascript
class InventoryTracker {
  updateStock(productId, quantity) {
    this.ws.send(JSON.stringify({
      type: 'stock-update',
      productId,
      quantity
    }));
  }

  onStockChange(callback) {
    this.ws.onmessage = (event) => {
      const data = JSON.parse(event.data);
      callback(data);
    };
  }
}
```

### 19. Live Code Execution (REPL)
```javascript
class LiveREPL {
  async executeCode(code, language) {
    const response = await fetch('/api/execute', {
      method: 'POST',
      body: JSON.stringify({ code, language })
    });
    const result = await response.json();
    this.displayOutput(result);
  }
}
```

### 20. Real-Time Order Tracking
```javascript
class OrderTracker {
  trackOrder(orderId) {
    this.ws = new WebSocket(`wss://orders.com/${orderId}`);
    this.ws.onmessage = (event) => {
      const status = JSON.parse(event.data);
      this.updateOrderStatus(status);
    };
  }
}
```

### 21. Live Video Call Stats
```javascript
class VideoCallStats {
  monitorConnection(peerConnection) {
    setInterval(async () => {
      const stats = await peerConnection.getStats();
      this.displayStats(stats);
    }, 1000);
  }
}
```

### 22. Real-Time Form Auto-Save
```javascript
class AutoSaveForm {
  constructor(formId) {
    this.form = document.getElementById(formId);
    this.setupAutoSave();
  }

  setupAutoSave() {
    this.form.addEventListener('input', this.debounce(() => {
      this.saveForm();
    }, 2000));
  }

  async saveForm() {
    const formData = new FormData(this.form);
    await fetch('/api/save', {
      method: 'POST',
      body: formData
    });
    this.showSaveIndicator();
  }
}
```

### 23. Live Calendar Sync
```javascript
class CalendarSync {
  syncEvents() {
    this.ws.onmessage = (event) => {
      const data = JSON.parse(event.data);
      if (data.type === 'event-created') {
        this.addEvent(data.event);
      } else if (data.type === 'event-updated') {
        this.updateEvent(data.event);
      }
    };
  }
}
```

### 24. Real-Time Task Queue Monitor
```javascript
class TaskQueueMonitor {
  constructor() {
    this.queue = [];
    this.processing = [];
    this.completed = [];
  }

  addTask(task) {
    this.queue.push(task);
    this.updateUI();
  }

  processTask(taskId) {
    const task = this.queue.find(t => t.id === taskId);
    this.queue = this.queue.filter(t => t.id !== taskId);
    this.processing.push(task);
    this.updateUI();
  }
}
```

### 25. Live Price Comparison
```javascript
class PriceComparison {
  async fetchPrices(productId) {
    const response = await fetch(`/api/prices/${productId}`);
    const prices = await response.json();
    
    this.displayPrices(prices);
    this.highlightBestDeal(prices);
  }

  highlightBestDeal(prices) {
    const lowest = Math.min(...prices.map(p => p.price));
    // Highlight the lowest price
  }
}
```

---

## Summary

These 25 examples cover:
- **WebSocket Communication**: Real-time bidirectional data
- **Event-Driven Architecture**: Handling live updates
- **Debouncing/Throttling**: Optimizing API calls
- **Progressive Enhancement**: Loading and updating data
- **State Management**: Tracking real-time changes
- **Error Handling**: Graceful failures and reconnection
- **Performance**: Efficient rendering and updates
- **User Experience**: Visual feedback and animations

---

**End of Real-Time Data Examples**