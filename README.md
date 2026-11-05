# Flying-Bird
It is a game which i made in this there is a bird which goes through the pillars with a endless loop and after every 15th pillar colour changes

HERE IS THE FULL CODE.

<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<meta name="viewport" content="width=device-width, initial-scale=1.0">
	<title>Flappy Bird</title>
	<style>
		* { box-sizing: border-box; }
		body {
			margin: 0;
			min-height: 100vh;
			display: grid;
			place-items: center;
			background: #18212b;
			font-family: system-ui, sans-serif;
		}
		canvas {
			display: block;
			width: min(100vw, 400px);
			height: auto;
			border: 4px solid #fff;
			border-radius: 12px;
			box-shadow: 0 12px 40px #0008;
			touch-action: manipulation;
		}
	</style>
</head>
<body>
	<canvas id="game" width="400" height="600" aria-label="Flappy Bird game"></canvas>
	<script>
		const canvas = document.getElementById('game');
		const ctx = canvas.getContext('2d');
		const W = canvas.width;
		const H = canvas.height;
		const groundHeight = 58;
		const bird = { x: 95, y: 270, radius: 15, velocity: 0 };
		const pipeWidth = 62;
		const pipeGap = 160;
		let pipes = [];
		let score = 0;
		let best = Number(localStorage.getItem('flappyBest') || 0);
		let state = 'ready';
		let lastTime = 0;
		let spawnTimer = 0;
		let introTime = 0;
		pipes = [
			{ x: 185, top: 145, scored: false },
			{ x: 395, top: 225, scored: false }
		];

		function reset() {
			bird.y = 270;
			bird.velocity = 0;
			pipes = [];
			score = 0;
			spawnTimer = 0;
			state = 'playing';
			flap();
		}

		function flap() {
			bird.velocity = -390;
		}

		function handleInput() {
			if (state === 'ready') reset();
			else if (state === 'over') reset();
			else flap();
		}

		document.addEventListener('keydown', event => {
			if (event.code === 'Space' || event.code === 'ArrowUp') {
				event.preventDefault();
				if (!event.repeat) handleInput();
			}
		});
		canvas.addEventListener('pointerdown', handleInput);

		function addPipe() {
			const minTop = 55;
			const maxTop = H - groundHeight - pipeGap - 55;
			const top = minTop + Math.random() * (maxTop - minTop);
			pipes.push({ x: W + 10, top, scored: false });
		}

		function endGame() {
			state = 'over';
			if (score > best) {
				best = score;
				localStorage.setItem('flappyBest', best);
			}
		}

		function update(dt) {
			if (state !== 'playing') return;
			bird.velocity += 1450 * dt;
			bird.y += bird.velocity * dt;

			spawnTimer += dt;
			if (spawnTimer >= 1.45) {
				spawnTimer = 0;
				addPipe();
			}

			for (const pipe of pipes) {
				pipe.x -= 175 * dt;
				if (!pipe.scored && pipe.x + pipeWidth < bird.x) {
					pipe.scored = true;
					score++;
				}

				const overlapsX = bird.x + bird.radius > pipe.x && bird.x - bird.radius < pipe.x + pipeWidth;
				if (overlapsX && (bird.y - bird.radius < pipe.top || bird.y + bird.radius > pipe.top + pipeGap)) {
					endGame();
				}
			}
			pipes = pipes.filter(pipe => pipe.x + pipeWidth > 0);

			if (bird.y - bird.radius < 0 || bird.y + bird.radius > H - groundHeight) endGame();
		}

		function drawBackground() {
			const sky = ctx.createLinearGradient(0, 0, 0, H);
			sky.addColorStop(0, '#72d5f7');
			sky.addColorStop(1, '#d7f6ff');
			ctx.fillStyle = sky;
			ctx.fillRect(0, 0, W, H);

			ctx.fillStyle = '#ffffff88';
			for (const cloud of [{ x: 62, y: 100 }, { x: 290, y: 175 }, { x: 180, y: 45 }]) {
				ctx.beginPath();
				ctx.arc(cloud.x, cloud.y, 22, 0, Math.PI * 2);
				ctx.arc(cloud.x + 23, cloud.y - 7, 28, 0, Math.PI * 2);
				ctx.arc(cloud.x + 49, cloud.y, 20, 0, Math.PI * 2);
				ctx.fill();
			}
		}

		function drawPipes() {
			const pipeThemes = [
				{ side: '#164b9a', light: '#4aa9ff', mid: '#1674dc', dark: '#104a9c', cap: '#1455ad', shine: '#8bd5ff' },
				{ side: '#86279b', light: '#f18aff', mid: '#c43bd4', dark: '#79248e', cap: '#982bad', shine: '#ffc0ff' },
				{ side: '#b34a16', light: '#ffc05c', mid: '#f18322', dark: '#a94b17', cap: '#c65b19', shine: '#ffe09a' },
				{ side: '#08756e', light: '#56f0c4', mid: '#14b99e', dark: '#08776e', cap: '#0b9580', shine: '#a1ffe1' },
				{ side: '#9b2751', light: '#ff86a6', mid: '#e33e72', dark: '#9b2852', cap: '#b52d5e', shine: '#ffc1d1' }
			];
			const theme = pipeThemes[Math.floor(score / 15) % pipeThemes.length];
			for (const pipe of pipes) {
				const bottomY = pipe.top + pipeGap;
				const lowerHeight = H - groundHeight - bottomY;
				// Dark side faces, bevels, and highlights create a glossy 3D look.
				ctx.fillStyle = theme.side;
				ctx.fillRect(pipe.x + pipeWidth - 10, 0, 10, pipe.top);
				ctx.fillRect(pipe.x + pipeWidth - 10, bottomY, 10, lowerHeight);
				const pipeGradient = ctx.createLinearGradient(pipe.x, 0, pipe.x + pipeWidth, 0);
				pipeGradient.addColorStop(0, theme.dark);
				pipeGradient.addColorStop(0.2, theme.light);
				pipeGradient.addColorStop(0.55, theme.mid);
				pipeGradient.addColorStop(1, theme.dark);
				ctx.fillStyle = pipeGradient;
				ctx.fillRect(pipe.x, 0, pipeWidth, pipe.top);
				ctx.fillRect(pipe.x, bottomY, pipeWidth, lowerHeight);
				ctx.fillStyle = theme.shine;
				ctx.fillRect(pipe.x + 8, 0, 7, pipe.top);
				ctx.fillRect(pipe.x + 8, bottomY, 7, lowerHeight);
				ctx.fillStyle = theme.cap;
				ctx.fillRect(pipe.x - 5, pipe.top - 20, pipeWidth + 10, 20);
				ctx.fillRect(pipe.x - 5, bottomY, pipeWidth + 10, 20);
				ctx.fillStyle = theme.shine;
				ctx.fillRect(pipe.x - 5, pipe.top - 20, pipeWidth + 10, 5);
				ctx.fillRect(pipe.x - 5, bottomY, pipeWidth + 10, 5);
				ctx.fillStyle = theme.side;
				ctx.fillRect(pipe.x + pipeWidth, pipe.top - 15, 5, 15);
				ctx.fillRect(pipe.x + pipeWidth, bottomY + 5, 5, 15);
			}
		}

		function drawBird() {
			ctx.save();
			ctx.translate(bird.x, bird.y);
			ctx.rotate(Math.max(-0.45, Math.min(0.8, bird.velocity / 650)));
			// Bright, playful plumage around a tiny Modi-inspired face.
			const feathers = ctx.createRadialGradient(-7, -9, 2, 1, 1, 22);
			feathers.addColorStop(0, '#fff3a1');
			feathers.addColorStop(0.42, '#ffd633');
			feathers.addColorStop(0.78, '#ff9b23');
			feathers.addColorStop(1, '#bd541d');
			ctx.fillStyle = feathers;
			ctx.beginPath();
			ctx.ellipse(0, 0, bird.radius + 3, bird.radius + 1, 0, 0, Math.PI * 2);
			ctx.fill();
			ctx.strokeStyle = '#fff4bd';
			ctx.lineWidth = 1.5;
			ctx.stroke();
			ctx.fillStyle = '#ffffff88';
			ctx.beginPath();
			ctx.ellipse(-5, -10, 5, 2, -0.4, 0, Math.PI * 2);
			ctx.fill();
			ctx.fillStyle = '#287ee8';
			ctx.beginPath();
			ctx.ellipse(-7, 6, 8, 5, -0.3, 0, Math.PI * 2);
			ctx.fill();
			// Cartoon face details: swept-back silver hair, beard and round glasses.
			ctx.fillStyle = '#d99a68';
			ctx.beginPath();
			ctx.arc(5, -3, 9, 0, Math.PI * 2);
			ctx.fill();
			ctx.fillStyle = '#f4f1e8';
			ctx.beginPath();
			ctx.ellipse(4, -10, 8, 4, -0.15, Math.PI, Math.PI * 2);
			ctx.fill();
			ctx.fillStyle = '#f4f1e8';
			ctx.beginPath();
			ctx.ellipse(5, 0, 7, 5, 0, 0, Math.PI);
			ctx.fill();
			ctx.strokeStyle = '#272b30';
			ctx.lineWidth = 1.4;
			ctx.beginPath();
			ctx.arc(2, -4, 2.4, 0, Math.PI * 2);
			ctx.arc(8, -4, 2.4, 0, Math.PI * 2);
			ctx.moveTo(4.4, -4);
			ctx.lineTo(5.6, -4);
			ctx.stroke();
			ctx.fillStyle = '#222';
			ctx.beginPath();
			ctx.arc(2, -4, 0.7, 0, Math.PI * 2);
			ctx.arc(8, -4, 0.7, 0, Math.PI * 2);
			ctx.fill();
			ctx.fillStyle = '#cf3038';
			ctx.beginPath();
			ctx.ellipse(5, -10, 1, 2, 0, 0, Math.PI * 2);
			ctx.fill();
			ctx.fillStyle = '#f07828';
			ctx.beginPath();
			ctx.moveTo(13, 1);
			ctx.lineTo(27, 5);
			ctx.lineTo(13, 9);
			ctx.closePath();
			ctx.fill();
			ctx.restore();
		}

		function drawGround() {
			const dirt = ctx.createLinearGradient(0, H - groundHeight, 0, H);
			dirt.addColorStop(0, '#f4e9a1');
			dirt.addColorStop(0.2, '#c7b76e');
			dirt.addColorStop(1, '#8a7545');
			ctx.fillStyle = dirt;
			ctx.fillRect(0, H - groundHeight, W, groundHeight);
			const grass = ctx.createLinearGradient(0, H - groundHeight, 0, H - groundHeight + 13);
			grass.addColorStop(0, '#b8f46a');
			grass.addColorStop(1, '#4b9b36');
			ctx.fillStyle = grass;
			ctx.fillRect(0, H - groundHeight, W, 13);
			ctx.fillStyle = '#b7a966';
			for (let x = 0; x < W; x += 34) {
				ctx.fillRect(x, H - 25, 18, 5);
				ctx.fillRect(x + 15, H - 42, 12, 4);
			}
		}

		function drawText(text, y, size = 28) {
			ctx.font = `bold ${size}px system-ui, sans-serif`;
			ctx.textAlign = 'center';
			ctx.lineWidth = 5;
			ctx.strokeStyle = '#603b22';
			ctx.fillStyle = '#fff';
			ctx.strokeText(text, W / 2, y);
			ctx.fillText(text, W / 2, y);
		}

		function draw() {
			drawBackground();
			drawPipes();
			drawGround();
			drawBird();

			if (state === 'playing') drawText(String(score), 65, 42);
			if (state === 'ready') {
				ctx.fillStyle = '#102536dd';
				ctx.beginPath();
				ctx.roundRect(32, 145, W - 64, 230, 20);
				ctx.fill();
				ctx.strokeStyle = '#ffffff';
				ctx.lineWidth = 3;
				ctx.stroke();
				ctx.fillStyle = '#8ee8ff';
				ctx.font = 'bold 13px system-ui, sans-serif';
				ctx.textAlign = 'center';
				ctx.fillText('READY TO FLY?', W / 2, 181);
				drawText('FLAPPY BIRD', 222, 36);
				drawText('Dodge the pipes and fly as far as you can!', 252, 14);
				ctx.fillStyle = '#f3a712';
				ctx.beginPath();
				ctx.roundRect(115, 275, 170, 50, 12);
				ctx.fill();
				ctx.strokeStyle = '#fff';
				ctx.lineWidth = 2;
				ctx.stroke();
				drawText('START', 308, 22);
				drawText('Tap / click or press SPACE', 353, 15);
				ctx.fillStyle = '#102536';
				ctx.font = 'bold 12px system-ui, sans-serif';
				ctx.textAlign = 'center';
				ctx.fillText('LOADING', W / 2, 557);
				ctx.fillStyle = '#ffffffaa';
				ctx.beginPath();
				ctx.roundRect(120, 566, 160, 9, 5);
				ctx.fill();
				ctx.fillStyle = '#f3a712';
				ctx.beginPath();
				ctx.roundRect(120, 566, 160 * (0.25 + 0.75 * ((Math.sin(introTime * 2) + 1) / 2)), 9, 5);
				ctx.fill();
			}
			if (state === 'over') {
				drawText('GAME OVER', 215, 34);
				drawText(`Score: ${score}   Best: ${best}`, 258, 20);
				drawText('Tap or press SPACE to retry', 295, 17);
			}
		}

		function loop(time) {
			const dt = Math.min((time - lastTime) / 1000 || 0, 0.033);
			lastTime = time;
			if (state === 'ready') {
				introTime += dt;
				bird.y = 270 + Math.sin(time / 250) * 7;
				for (const pipe of pipes) pipe.x -= 100 * dt;
				pipes = pipes.filter(pipe => pipe.x + pipeWidth > 0);
				if (!pipes.length || pipes[pipes.length - 1].x < W - 210) {
					const minTop = 55;
					const maxTop = H - groundHeight - pipeGap - 55;
					pipes.push({ x: W + 10, top: minTop + Math.random() * (maxTop - minTop), scored: false });
				}
			}
			update(dt);
			draw();
			requestAnimationFrame(loop);
		}

		requestAnimationFrame(loop);
	</script>
</body>
</html>
