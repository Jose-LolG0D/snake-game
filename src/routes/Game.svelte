<script lang="ts">
	enum Direction {
		Up,
		Down,
		Left,
		Right
	}

	var debug = false;

	//Game dimensions
	const max_x = 10;
	const max_y = 10;

	//Game variables
	var game = [];
	var game_over = false;
	var apple_position;
	var speed = 250;
	var ate_apple = false;
	var game_loop;
	var apple_record = 0;

	//Snake Object
	var snake = {
		position: {
			x: Math.ceil((max_x - 1) / 2),
			y: Math.ceil((max_y - 1) / 2)
		},
		direction: Direction.Right,
		head_position: {
			x: Math.ceil((max_x - 1) / 2),
			y: Math.ceil((max_y - 1) / 2)
		},
		tail_positions: []
	};

	function position(x, y, direction) {
		return {
			x: x,
			y: y,
			direction: direction
		};
	}

	function getRandomIntInclusive(max) {
		max = Math.floor(max);
		return Math.floor(Math.random() * (max + 1));
	}

	//get position for apple, position can't be snake neither tail
	function getPositionForApple() {
		var x = getRandomIntInclusive(max_x - 1);
		var y = getRandomIntInclusive(max_y - 1);
		while (game[x][y] === 'snake' || game[x][y] === 'tail') {
			x = getRandomIntInclusive(max_x - 1);
			y = getRandomIntInclusive(max_y - 1);
		}
		return position(x, y, Direction.Up);
	}

	function initializeGame() {
		game_over = false;
		//fill 2d game array
		for (let x = 0; x < max_x; x++) {
			game[x] = [];
			for (let y = 0; y < max_y; y++) {
				game[x][y] = '(' + x + ', ' + y + ')';
			}
		}
		//initialize snake object
		snake = {
			position: {
				x: Math.ceil((max_x - 1) / 2),
				y: Math.ceil((max_y - 1) / 2)
			},
			direction: Direction.Right,
			head_position: {
				x: Math.ceil((max_x - 1) / 2),
				y: Math.ceil((max_y - 1) / 2)
			},
			tail_positions: []
		};
		//put the snake, the tail and the apple in the map
		snake.tail_positions.push(position(snake.position.x - 1, snake.position.y, Direction.Right));
		game[snake.tail_positions[0].x][snake.tail_positions[0].y] = 'tail';

		game[snake.position.x][snake.position.y] = 'snake';

		apple_position = getPositionForApple();
		game[apple_position.x][apple_position.y] = 'apple';
	}

	function startGame() {
		game_loop = setInterval(updateGame, speed);
	}

	function reset() {
		next_direction = Direction.Right;
		clearInterval(game_loop);
		initializeGame();
		startGame();
	}

	function end() {
		game_over = true;
		clearInterval(game_loop);

		if (snake.tail_positions.length - 1 > apple_record)
			apple_record = snake.tail_positions.length - 1;
	}

	let next_direction = Direction.Right;

	function handleKeydown(event) {
		let keyCode = event.keyCode;
		switch (keyCode) {
			case 38:
				if (snake.direction !== Direction.Down) next_direction = Direction.Up;
				break;
			case 40:
				if (snake.direction !== Direction.Up) next_direction = Direction.Down;
				break;
			case 39:
				if (snake.direction !== Direction.Left) next_direction = Direction.Right;
				break;
			case 37:
				if (snake.direction !== Direction.Right) next_direction = Direction.Left;
				break;
			default:
				break;
		}
	}

	function updateGame() {
		snake.direction = next_direction;

		//only remove last tail square if player didnt eat apple, otherwise, skip removing snake tail for one iteration
		if (!ate_apple) {
			//remove last tail square from list, update game, and add new tail square where snake head was
			let last_tail = snake.tail_positions.shift();
			game[last_tail.x][last_tail.y] = '(' + last_tail.x + ', ' + last_tail.y + ')';
		}

		ate_apple = false;
		snake.tail_positions.push(position(snake.position.x, snake.position.y, snake.direction));

		switch (snake.direction) {
			case Direction.Down:
				if (snake.position.y - 1 >= 0) snake.position.y--;
				else end();
				break;
			case Direction.Up:
				if (snake.position.y + 1 <= max_y - 1) snake.position.y++;
				else end();
				break;
			case Direction.Right:
				if (snake.position.x + 1 <= max_x - 1) snake.position.x++;
				else end();
				break;
			case Direction.Left:
				if (snake.position.x - 1 >= 0) snake.position.x--;
				else end();
				break;
		}

		if (game[snake.position.x][snake.position.y] === 'tail') {
			end();
			return;
		}

		game[snake.position.x][snake.position.y] = 'snake';

		for (let position of snake.tail_positions) {
			game[position.x][position.y] = 'tail';
		}

		//check if snake is going to a square where a apple is located
		if (snake.position.x === apple_position.x && snake.position.y === apple_position.y) {
			ate_apple = true;
			apple_position = getPositionForApple();
			game[apple_position.x][apple_position.y] = 'apple';
		}
	}

	initializeGame();
</script>

<svelte:window on:keydown={handleKeydown} />

{#if game_loop === undefined}
	<div class="overlay">
		<button
			on:click={() => {
				game_loop === undefined ? startGame() : reset();
			}}>Start</button
		>
	</div>
{:else if game_over === true}
	<div class="overlay">
		<div class="stats">
			<p>
				<img src="/apple.png" alt="Apples eaten" width="20px" />{snake.tail_positions.length - 1}
			</p>
			<p>
				<img src="/trofeu.png" alt="Trophy" width="20px" />{apple_record}
			</p>
		</div>
		<button
			on:click={() => {
				game_loop === undefined ? startGame() : reset();
			}}>Try again</button
		>
	</div>
{/if}

<div class="game">
	<div class="rows">
		{#each game as row}
			<div class="collumns">
				{#each row as collumn}
					<div
						class="square {collumn == 'snake'
							? 'snakedirection-' + snake.direction.toString()
							: ''}"
						class:snake={collumn == 'snake'}
						class:tail={collumn == 'tail'}
						class:apple={collumn == 'apple'}
					>
						{#if debug}
							{collumn}
						{/if}
					</div>
				{/each}
			</div>
		{/each}
	</div>
</div>

<style>
	.collumns {
		display: flex;
		flex-direction: column-reverse;
	}

	.square {
		height: 50px;
		width: 50px;
		background-color: var(--square-color);
		border: rgb(0, 110, 255) solid 1px;
	}
	.rows {
		display: flex;
	}

	.snake {
		background-color: green;
	}

	.tail {
		background-color: goldenrod;
	}

	.apple {
		background-color: rgb(255, 38, 0);
	}

	.overlay {
		position: absolute;
		display: flex;
		justify-content: center;
		align-items: center;
		flex-direction: column;
		background-color: rgba(128, 128, 128, 0.2);
		width: 100vw;
		height: 100vh;
	}

	.stats {
		display: flex;
	}

	.stats p:first-child {
		padding-right: 10px;
	}

	button {
		padding: 5px 20px 5px 20px;
		border: solid 1px white;
		border-radius: 5px;
	}

	button:active {
		background-color: gray;
	}
</style>
