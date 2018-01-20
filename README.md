<!DOCTYPE html>
<html>

	<head>
		<meta charset="UTF-8">
		<title>猪头大作战</title>
		<style>
			#myCanvas {
				background: black;
				display: block;
				margin: 0 auto;
			}
		</style>
	</head>

	<body>
		<canvas id="myCanvas">
			您的浏览器版本过低，请您更新版本
		</canvas>
	</body>
	<script>
		var myCanvas = document.getElementById('myCanvas'); //画布
		var ctx = myCanvas.getContext('2d'); //画笔

		myCanvas.width = 800;
		myCanvas.height = 800;

		//鼠标基于canvas画布的x坐标（绘制白色块及子弹）
		var mouseX = myCanvas.width / 2;

		//分数，初始值为0
		var score = 0;
		//子弹数组
		var bulletArr = [];
		//掉落快数组
		var enemyArr = [];
		//爆炸物数组
		var boomArr = [];
		//没被击中，且移出画布的掉落物的数量，根据此值判断是否结束游戏
		var dieNum = 0;

		//
		var eleMoveTimer;
		var createEnemyTimer;

		//随机数函数
		function rn(x, y) {
			return Math.round(Math.random() * (y - x) + x);
		}

		function drawStratPage() {
			ctx.beginPath();
			ctx.font = '60px Arial';
			ctx.fillStyle = '#fff';
			ctx.fillText('🐷️ 大作战', 250, 400);

			ctx.beginPath();
			ctx.font = '30px Arial';
			ctx.fillStyle = '#F5F5F5';
			ctx.fillText('点击游戏页面任何位置开始游戏', 180, 450);
		}
		drawStratPage();

		//白块函数
		function drawBox() {
			//判断盒子x值的最大值，不让盒子从画布出去
			if(mouseX > myCanvas.width - 20) {
				mouseX = myCanvas.width - 20;
			}
			//判断盒子X值的最小值（只能用if语句，不能用三目）

			mouseX = mouseX < 20 ? 20 : mouseX;
			//			if(mouseX < 20){
			//				mouseX = 20;
			//			}
			ctx.beginPath();
			//ctx.fillStyle = 'transparent';
			//ctx.fillRect(mouseX - 20, 730, 40, 40);
			ctx.font = '60px Arial';
			ctx.fillText('🐷', mouseX - 30, 760);
			ctx.fill;
		}

		//封装子弹类
		function Bullet() {
			this.x = mouseX - 4; //子弹X坐标
			this.y = 722;
			this.speed = 2; //速度，子弹匀速运动
			ctx.beginPath();
			ctx.fillStyle = '#fff';
			ctx.fillRect(this.x, this.y, 8, 8);
			ctx.fill();
		}

		Bullet.prototype.move = function() {
			this.y -= this.speed;
			ctx.beginPath();
			ctx.fillStyle = '#fff';
			ctx.fillRect(this.x, this.y, 8, 8);
			ctx.fill();
			//当子弹移出画布时，从数组中删除，减少不必要的循环，出去的子弹是数组中的第一个元素
			if(this.y < 0) {
				bulletArr.shift();
			}
		}

		//创建一个子弹
		function createBullet() {
			var bullet = new Bullet(); //实例化子弹
			bulletArr.push(bullet); //将子弹放到子弹数组	
		}

		/*
		 Ememy类： 掉落块的类
		 x,y 左上角的坐标
		 vx，vy 水平方向和竖直方向移动的速度
		 bc 背景色
		 dis 左右摇摆的范围
		 */
		function Enemy(x, wh, vx, vy, bc, dis) {
			this.x = x;
			this.y = -wh;
			this.wh = wh;
			this.vx = vx;
			this.vy = vy;
			this.bc = bc;
			this.dis = dis;
			this.left = this.x - dis; //摆动左边边界
			this.right = this.x + dis; //摆动右边的边界
		}
		Enemy.prototype.move = function() {
			//当块左右摆动到达边界之后，反弹
			if(this.x < this.left || this.x > this.right || this.x < 0 || this.x > myCanvas.width - this.wh) {
				this.vx *= -1;
			}
			this.x += this.vx;
			this.y += this.vy;
			ctx.beginPath();
			ctx.fillStyle = this.bc;
			ctx.fillRect(this.x, this.y, this.wh, this.wh);
			ctx.fill();
		}

		//实例化enemy对象函数
		var minWh = 40,
			maxWh = 70; //宽高范围
		var minX = 0,
			maxX = myCanvas.width - maxWh; //x坐标范围
		var minVx = -2,
			maxVx = 3; //x方向速度范围
		var minVy = 1,
			maxVy = 3; //y方向速度范围
		var minDis = 100,     
			maxDis = 200; //摆动范围
		function createEnemy() {
			var x = rn(minX, maxX);
			var wh = rn(minWh, maxWh);
			var vx = rn(minVx, maxVx);
			var vy = rn(minVy, maxVy);
			var dis = rn(minDis, maxDis);
			var bc = 'rgb(' + rn(100, 255) + ',' + rn(100, 255) + ',' + rn(100, 255) + ')';
			//实例化一个掉落块
			var enemy = new Enemy(x, wh, vx, vy, bc, dis);
			enemyArr.push(enemy);
		}

		//判断小块是否移出画布，移出画布从enemy数组中删除
		function judgeEnemy() {
			for(var i = 0; i < enemyArr.length; i++) {
				if(enemyArr[i].y > myCanvas.height) {
					enemyArr.splice(i, 1);
					dieNum++;
					//break;			
					//移出元素之后数组结构发生变化，为了放止漏判，要让i-1                                                                                     
					i--;
				}
			}
		}

		//封装爆炸物类
		function Boom(x, y, vx, vy, bc) {
			this.x = x; //x坐标
			this.y = y; //y坐标
			this.vx = vx; //水平方向速度
			this.vy = vy; //垂直方向速度
			this.bc = bc; //背景色
			this.times = 0; //爆炸物的绘制次数（move函数每调一次，time加+）
		}
		Boom.prototype.move = function() {
			this.x += this.vx;
			this.y += this.vy;
			ctx.beginPath();
			ctx.fillStyle = this.bc;
			ctx.fillRect(this.x, this.y, 4, 4);
			ctx.fill();
			this.times++;
		}

		//判断是否击中(碰撞检测)
		function judgeHit() {
			for(var i = 0; i < bulletArr.length; i++) {
				for(var j = 0; j < enemyArr.length; j++) {
					var a = bulletArr[i];
					var b = enemyArr[j];
					//a和b的碰撞检测
					if(a.x + 8 > b.x && a.y + 8 > b.y && a.x < b.x + b.wh && a.y < b.y + b.wh) {
						//分数累加
						score += 10;
						//根据被击中块的信息创建爆炸物
						createBoom(b.x, b.y, b.wh, b.bc);
						//两两碰撞
						bulletArr.splice(i, 1); //移除子弹
						enemyArr.splice(j, 1); //移除掉落物
						i--;
						//当碰撞上时,两个块都已经没有作用,删除之后,不用做多余的比较,直接跳出内层循环,让外层循环进行下一次碰撞检测
						break;
					}
				}
			}
		}

		//创建爆炸物
		function createBoom(x, y, wh, bc) {
			var nowArr = []; //临时数组,放当前被击中的块产生的那一批爆炸物
			//实例化boom
			var num = Math.ceil(wh / 4);
			//双层循环实例化类
			for(var i = 0; i < num; i++) {
				//计算每行的y坐标
				var thisY = y + 8 * i;
				for(var j = 0; j < num; j++) {
					var thisX = x + 8 * j;
					//实例化爆炸物对象
					var vx = rn(-4, 4);
					var vy = rn(-4, 4);
					if(vx == 0 && vy == 0) {
						vx = -4;
						vy = 4
					}
					var boom = new Boom(thisX, thisY, vx, vy, bc);
					nowArr.push(boom);
				}
			}
			boomArr.push(nowArr);
		}

		//判断爆炸物
		function judgeBoom() {
			for(var i = 0; i < boomArr.length; i++) {
				for(var j = 0; j < boomArr[i].length; j++) {
					//判断一批爆炸物中移动最慢的是否从画布移除
					var maxTimes = Math.ceil(Math.sqrt(Math.pow(myCanvas.width, 2) + Math.pow(myCanvas.height, 2)));
					if(boomArr[i][j].times > maxTimes) {
						boomArr.splice(i, 1);
						i--;
						break;
					}
				}
			}
		}

		//绘制分数
		function drawScore() {
			ctx.beginPath();
			ctx.font = '20px Arial';
			ctx.fillStyle = 'red';
			ctx.fillText('分数：' + score, 20, 20);
		}

		//点击事件的开关  0：开始游戏 ； 1：绘制子弹 ；2：游戏结束
		var gameFlag = 0;
		//为canvas绑定点击事件
		myCanvas.onclick = function() {
			//第一次点击画布，开始游戏，之后再点击画布，创建子弹
			if(gameFlag == 0) {
				//为canvas添加鼠标移动事件Ï
				myCanvas.onmousemove = function() {
					//ctx.clearRect(0,0,800,800);
					//计算出鼠标基于canvas的x坐标
					var e = window.event || event;
					mouseX = e.clientX - myCanvas.offsetLeft;
					//鼠标移动时要绘制白色快
					drawBox();
				}
				gamestart();
				gameFlag = 1;
				drawScore();
			} else if(gameFlag == 1) {
				createBullet(); //创建子弹
			} else {
				var e = window.event || event;
				if(e.pageX - myCanvas.offsetLeft > 330 && e.pageX - myCanvas.offsetLeft < 470 && e.pageY - myCanvas.offsetTop > 550 && e.pageY - myCanvas.offsetTop < 610) {
					gameFlag = 0;
					//点击canvas回到首页
					ctx.clearRect(0, 0, 800, 800);
					drawStratPage();
				}

			}
		}

		function gamestart() {

			//绘制频率
			eleMoveTimer = setInterval(function() {
				//根据重绘频率，清除一次画布
				ctx.clearRect(0, 0, 800, 800);

				//循环绘制所有子弹
				for(var i = 0; i < bulletArr.length; i++) {
					bulletArr[i].move();
				}
				//循环绘制掉落物
				for(var j = 0; j < enemyArr.length; j++) {
					enemyArr[j].move();
				}
				//绘制爆炸物
				for(var m = 0; m < boomArr.length; m++) {
					for(var n = 0; n < boomArr[m].length; n++) {
						boomArr[m][n].move();
					}
				}
				judgeEnemy(); //判断掉落物是否移出画布
				judgeHit(); //碰撞
				judgeBoom(); //判断爆炸物是否移出页面
				drawBox(); //绘制白块
				drawScore(); //绘制分数

				//检测游戏结束
				if(dieNum == 10) {
					gameOver();
				}
			}, 5);

			//生成掉落物的计时器
			createEnemyTimer = setInterval(function() {
				createEnemy();
			}, 500);
		}

		//游戏结束函数
		function gameOver() { //清除游戏运行计时器
			clearInterval(eleMoveTimer);
			clearInterval(createEnemyTimer);
			//清除画布的事件
			//myCanvas.onclick = null;
			myCanvas.onmousemove = null;
			ctx.clearRect(0, 0, 800, 800);
			//绘制最终分数
			ctx.beginPath();
			ctx.font = '50px Arial';
			ctx.fillStyle = 'yellow';
			ctx.fillText('最终得分：' + score, 250, 400);

			ctx.beginPath();
			ctx.font = '40px Arial';
			ctx.fillStyle = 'red';
			ctx.fillText('点击返回', 330, 550);

			//初始化所有数值
			score = 0;
			dieNum = 0;
			bulletArr = [];
			boomArr = [];
			enemyArr = [];

			gameFlag = 2;
		}
	</script>

</html>
