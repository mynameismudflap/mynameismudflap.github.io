
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Days Since Last Incident</title>
  <style>
    body {
      display: flex;
      flex-direction: column;
      align-items: center;
      height: 100vh;
      margin: 0;
      font-family: Arial, sans-serif;
      background-color: #f4f4f4;
    }
    h1 {
      margin-top: 60px;
      font-size: 2em;
    }
    .counter {
      font-size: 20em;
      margin: 0.2em 0 0.1em 0;
    }
    .button-group {
      display: flex;
      gap: 1em;
      margin: 2em 0;
      justify-content: center;
    }
    .btn {
      padding: 0.5em 1.5em;
      font-size: 1.2em;
      background-color: #007bff;
      color: white;
      border: none;
      border-radius: 8px;
      cursor: pointer;
      transition: background-color 0.3s ease;
    }
    .btn:hover {
      background-color: #0056b3;
    }
    .timestamp, .clock, .countdown, .reset-count {
      font-size: 1em;
      color: #555;
      margin: 5px 0;
    }
    .date-info {
      margin-top: 50px;
      display: flex;
      flex-direction: column;
      align-items: center;
    }
    .modal-overlay {
      position: fixed;
      top: 0;
      left: 0;
      width: 100vw;
      height: 100vh;
      background: rgba(0,0,0,0.5);
      display: none;
      align-items: center;
      justify-content: center;
      opacity: 0;
      transition: opacity 0.3s ease;
    }
    .modal-overlay.show {
      display: flex !important;
      opacity: 1;
    }
    .modal {
      background: white;
      padding: 2em;
      border-radius: 10px;
      text-align: center;
      max-width: 300px;
    }
    .modal button {
      margin: 1em 0.5em 0 0.5em;
      padding: 0.5em 1em;
      font-size: 1em;
      border: none;
      border-radius: 6px;
      cursor: pointer;
    }
    .yes {
      background-color: #dc3545;
      color: white;
    }
    .no {
      background-color: #6c757d;
      color: white;
    }
  </style>
</head>
<body>
  <h1>Days since last incident</h1>
  <div class="counter" id="counter">0</div>
  <div class="button-group">
    <button class="btn" onclick="openModal()">Reset</button>
  </div>
  <div class="date-info">
    <h4 class="timestamp" id="timestamp"></h4>
    <h4 class="reset-count" id="resetCount"></h4>
    <h4 class="clock" id="clock"></h4>
    <h4 class="countdown" id="countdown"></h4>
  </div>

  <div class="modal-overlay" id="modal" onclick="handleBackgroundClick(event)">
    <div class="modal">
      <p>Are you sure you want to reset the counter?</p>
      <button class="yes" onclick="confirmReset()">I messed up bad</button>
      <button class="no" onclick="closeModal()">Nevermind</button>
    </div>
  </div>

  <script>
    const counterElement = document.getElementById('counter');
    const timestampElement = document.getElementById('timestamp');
    const clockElement = document.getElementById('clock');
    const countdownElement = document.getElementById('countdown');
    const resetCountElement = document.getElementById('resetCount');
    const modal = document.getElementById('modal');

    let count = parseInt(localStorage.getItem('incidentCounter')) || 0;
    let lastUpdated = localStorage.getItem('incidentTimestamp');
    let lastIncrementDate = localStorage.getItem('lastIncrementDate');
    let resetCount = parseInt(localStorage.getItem('incidentResetCount')) || 0;

    counterElement.textContent = count;
    if (lastUpdated) timestampElement.textContent = `Last reset: ${formatDate(lastUpdated)}`;
    resetCountElement.textContent = `Total resets: ${resetCount}`;

    function formatDate(dateString) {
      const locale = navigator.language || 'en-US';
      const options = { dateStyle: 'medium', timeStyle: 'short' };
      return new Date(dateString).toLocaleString(locale, options);
    }

    function updateTimestamp() {
      const now = new Date();
      localStorage.setItem('incidentTimestamp', now);
      timestampElement.textContent = `Last reset: ${formatDate(now)}`;
    }

    function incrementIfNewDay() {
      const today = new Date().toDateString();
      if (lastIncrementDate !== today) {
        count++;
        localStorage.setItem('incidentCounter', count);
        localStorage.setItem('lastIncrementDate', today);
        counterElement.textContent = count;
        updateTimestamp();
      }
    }

    function updateClockAndCountdown() {
      const now = new Date();
      clockElement.textContent = `Current time: ${now.toLocaleTimeString()}`;

      const nextMidnight = new Date();
      nextMidnight.setHours(24, 0, 0, 0);
      const diff = nextMidnight - now;

      const hours = Math.floor(diff / (1000 * 60 * 60));
      const minutes = Math.floor((diff / (1000 * 60)) % 60);
      const seconds = Math.floor((diff / 1000) % 60);

      countdownElement.textContent = `Next day starts in: ${hours}h ${minutes}m ${seconds}s`;
    }

    setInterval(updateClockAndCountdown, 1000);

    function openModal() {
      modal.style.display = 'flex';
      requestAnimationFrame(() => modal.classList.add('show'));
    }

    function closeModal() {
      modal.classList.remove('show');
      setTimeout(() => { modal.style.display = 'none'; }, 300);
    }

    function confirmReset() {
      count = 0;
      resetCount++;
      counterElement.textContent = count;
      resetCountElement.textContent = `Total resets: ${resetCount}`;
      localStorage.setItem('incidentCounter', count);
      localStorage.setItem('incidentResetCount', resetCount);
      localStorage.setItem('lastIncrementDate', new Date().toDateString());
      updateTimestamp();
      closeModal();
    }

    function handleBackgroundClick(event) {
      if (event.target === modal) {
        closeModal();
      }
    }

    document.addEventListener('keydown', function(e) {
      if (modal.classList.contains('show')) {
        if (e.key === 'Escape') {
          closeModal();
        } else if (e.key === 'Enter') {
          confirmReset();
        }
      }
    });

    incrementIfNewDay();
  </script>
</body>
</html>
