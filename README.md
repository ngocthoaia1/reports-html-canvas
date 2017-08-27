Trong bài viết này tôi sẽ hướng dẫn tạo một game đơn giản mô phỏng chuyển động với canvas và javascript

Đầu tiên ta sẽ tạo file html và một vẽ ra một khung mô phỏng game bằng canvas với kích thước là 600 * 400

```
<!DOCTYPE html>
<html>
  <head>
    <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
    <style>
      canvas {
        border:1px solid #d3d3d3;
        background-color: #f1f1f1;
      }
    </style>
    <script src="http://ajax.googleapis.com/ajax/libs/jquery/1.7.1/jquery.min.js" type="text/javascript"></script>
  </head>
  <body onload="startGame()">
    <script>
      function startGame() {
        myGameArea.start();
      }

      var myGameArea = {
        canvas : document.createElement("canvas"),
        start : function() {
          this.canvas.width = 600;
          this.canvas.height = 400;
          this.context = this.canvas.getContext("2d");
          document.body.insertBefore(this.canvas, document.body.childNodes[0]);
        },
        clear : function() {
          this.context.clearRect(0, 0, this.canvas.width, this.canvas.height);
        }
      }
    </script>
  </body>
</html>

```

Tiếp theo sẽ thêm các đối tượng vào trong khung này. Đầu tiên sẽ thêm đối tượng mà người chơi điều khiển là một khối hình vuông

```javascript
var myGameArea = {
  canvas : document.createElement("canvas"),
  start : function() {
    this.canvas.width = 600;
    this.canvas.height = 400;
    this.context = this.canvas.getContext("2d");
    myGameArea.myGameObject = new gameObject(30, 30, "red", 10, 120);
    document.body.insertBefore(this.canvas, document.body.childNodes[0]);
    this.interval = setInterval(this.update, 20);
  },
  clear : function() {
    this.context.clearRect(0, 0, this.canvas.width, this.canvas.height);
  },
  update : function() {
    myGameArea.clear();
    myGameArea.myGameObject.update();
  }
}

function gameObject(width, height, color, x, y) {
  this.width = width;
  this.height = height;
  this.x = x;
  this.y = y;
  this.update = function() {
    ctx = myGameArea.context;
    ctx.fillStyle = color;
    ctx.fillRect(this.x, this.y, this.width, this.height);
  }
}
```
Ta đã thêm một đối tượng vào trong game. Giờ sẽ thêm phần mô phỏng chuyển động với tương tác di chuyển đối tượng này bằng các phím lên xuống trái phải. Khi di chuyển đối tượng ra ngoài biên thì game sẽ kết thúc!
```
function gameObject(width, height, color, x, y) {
  this.width = width;
  this.height = height;
  this.speedX = 0;
  this.speedY = 0;
  this.x = x;
  this.y = y;
  this.update = function() {
    ctx = myGameArea.context;
    ctx.fillStyle = color;
    this.move();
    ctx.fillRect(this.x, this.y, this.width, this.height);
  }

  this.move = function() {
    if (this.isDied) {return}
    this.x += this.speedX;
    this.y += this.speedY;
    if (this.speedY > 0) {
      this.speedY -= 0.02;
    } else {
      this.speedY += 0.02
    }
    if (this.speedX > 0) {
      this.speedX -= 0.02;
    } else {
      this.speedX += 0.02
    }
    this.checkCrash();
  }
  this.top = function() {return this.y;}
  this.bottom = function() {return this.y + this.height;}
  this.left = function() {return this.x;}
  this.right = function() {return this.x + this.width;}

  this.moveOutOfArea = function() {
    if (
      (this.top() < 0) || (this.bottom() > myGameArea.canvas.height) ||
      (this.left() < 0) || (this.right() > myGameArea.canvas.width)
    ) {
      return true;
    }
    return false;
  }

  this.checkCrash = function() {
    if (this.moveOutOfArea()) {
      this.isDied = true;
      return;
    }
  }
}

$(document).keydown(function (e) {
  var keyInput = e.which;
  var KEYS = {
    UP: 38,
    DOWN: 40,
    LEFT: 37,
    RIGHT: 39
  }
  switch (keyInput) {
    case KEYS.UP:
      myGameArea.myGameObject.speedY -= 0.5;
      e.preventDefault();
      break;
    case KEYS.DOWN:
      myGameArea.myGameObject.speedY += 0.5;
      e.preventDefault();
      break;
    case KEYS.LEFT:
      myGameArea.myGameObject.speedX -= 0.5;
      e.preventDefault();
      break;
    case KEYS.RIGHT:
      myGameArea.myGameObject.speedX += 0.5;
      e.preventDefault();
      break;
  }
});
```

Ta đã hoàn thành được một nửa chặng đường rồi. Giờ ta sẽ thêm các đối tượng khác là những bức tường. Thêm điều kiện kết thúc game là khi đối tượng mà người chơi điều khiển chạm vào các bức tường.

```
var myGameArea = {
  start : function() {
    ...
    myGameArea.myGameObject = new gameObject(30, 30, "red", 10, 120);
    myGameArea.frameNo = 0;
    myGameArea.obstacles = [];
    ...
  },
  update : function() {
    if (myGameArea.myGameObject.isDied) {
      return;
    }
    myGameArea.clear();
    myGameArea.frameNo += 1;
    if (myGameArea.frameNo % 150 == 1) {myGameArea.addObstacle()}
    for (i = 0; i < myGameArea.obstacles.length; i += 1) {
      myGameArea.obstacles[i].x += -1;
      myGameArea.obstacles[i].update();
    }
    myGameArea.myGameObject.move();
    myGameArea.myGameObject.update();
  },
  addObstacle : function() {
    canvasWidth = myGameArea.canvas.width;
    canvasHeight = myGameArea.canvas.height;
    minHeight = 20;
    maxHeight = 200;
    height = Math.floor(Math.random()*(maxHeight-minHeight+1)+minHeight);
    minGap = 50;
    maxGap = 200;
    gap = Math.floor(Math.random()*(maxGap-minGap+1)+minGap);
    myGameArea.obstacles.push(new gameObject(10, height, "green", canvasWidth, 0));
    myGameArea.obstacles.push(
      new gameObject(10, canvasHeight - height - gap, "green", canvasWidth, height + gap));
  }
}

function gameObject(width, height, color, x, y) {
  ...
  this.crashWith = function(other) {
    var crash = true;
    if (
      (this.bottom() < other.top()) || (this.top() > other.bottom()) ||
      (this.right() < other.left()) || (this.left() > other.right())
    ) {
      crash = false;
    }
    return crash;
  }

  this.checkCrash = function() {
    if (this.moveOutOfArea()) {
      this.isDied = true;
      return;
    }
    for (i = 0; i < myGameArea.obstacles.length; i += 1) {
      if (this.crashWith(myGameArea.obstacles[i])) {
        this.isDied = true;
      }
    }
  }
}
```
Tiếp theo ta thêm phần hiển thị điểm và hiển thị text khi game kết thúc.
```
var myGameArea = {
  start : function() {
    ...
    myGameArea.obstacles = [];
    myGameArea.score = new textComponent("30px", "Consolas", "black", 380, 40, "text");
    ...
  },
  update : function() {
    if (myGameArea.myGameObject.isDied) {
      var endedGameText = new textComponent("30px", "Consolas", "red", 200, 200, "text");
      endedGameText.text = "GAME OVER!";
      endedGameText.update()
      return;
    }
    myGameArea.clear();
    myGameArea.frameNo += 1;
    if (myGameArea.frameNo % 150 == 1) {myGameArea.addObstacle()}
    for (i = 0; i < myGameArea.obstacles.length; i += 1) {
      myGameArea.obstacles[i].x += -1;
      myGameArea.obstacles[i].update();
    }
    myGameArea.myGameObject.move();
    myGameArea.myGameObject.update();
    myGameArea.score.text="SCORE: " + myGameArea.frameNo;
    myGameArea.score.update();
  },
}

function textComponent(fontSize, font, color, x, y, ) {
  this.update = function() {
    ctx = myGameArea.context;
    ctx.font = fontSize + " " + font;
    ctx.fillStyle = color;
    ctx.fillText(this.text, x, y);
  }
}
```
Như vậy là ta đã có một game hoàn chỉnh. Bạn có thể thấy nó khá đơn giản với Canvas và javascript. Bạn có thể tùy chỉnh thêm hiệu ứng độ khó thay đổi cách tính điểm một cách dễ dàng.
