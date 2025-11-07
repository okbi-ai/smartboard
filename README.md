<!DOCTYPE html>
<html lang="ar">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>السبورة الذكية</title>
  <style>
    body {
      font-family: 'Tahoma', sans-serif;
      text-align: center;
      background-color: #f4f4f4;
      margin: 0;
      padding: 20px;
    }
    h1 {
      color: #3b5998;
    }
    canvas {
      border: 2px solid #3b5998;
      background: white;
      margin-top: 20px;
      cursor: crosshair;
    }
    button {
      background-color: #3b5998;
      color: white;
      border: none;
      padding: 10px 15px;
      margin: 10px;
      border-radius: 6px;
      cursor: pointer;
    }
    button:hover {
      background-color: #2d4373;
    }
  </style>
</head>
<body>
  <h1>🧠 السبورة الذكية</h1>
  <canvas id="board" width="600" height="400"></canvas><br>
  <button onclick="clearBoard()">مسح</button>
  <button onclick="saveImage()">حفظ الصورة</button>

  <script>
    const canvas = document.getElementById('board');
    const ctx = canvas.getContext('2d');
    let drawing = false;

    canvas.addEventListener('mousedown', () => drawing = true);
    canvas.addEventListener('mouseup', () => drawing = false);
    canvas.addEventListener('mousemove', draw);

    function draw(e) {
      if (!drawing) return;
      ctx.lineWidth = 2;
      ctx.lineCap = 'round';
      ctx.strokeStyle = '#000';
      ctx.lineTo(e.offsetX, e.offsetY);
      ctx.stroke();
      ctx.beginPath();
      ctx.moveTo(e.offsetX, e.offsetY);
    }

    function clearBoard() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
    }

    function saveImage() {
      const link = document.createElement('a');
      link.download = 'smartboard.png';
      link.href = canvas.toDataURL();
      link.click();
    }
  </script>
</body>
</html>
