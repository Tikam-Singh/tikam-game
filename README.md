import turtle
import random

# Constants
w = 1000
h = 600
food_size = 15
initial_delay = 100
min_delay = 60
level_increment = 10
score = 0
level = 1
delay = initial_delay  # Define delay here

# Define wall boundaries
left_boundary = -w / 2 + 20
right_boundary = w / 2 - 20
top_boundary = h / 2 - 20
bottom_boundary = -h / 2 + 20

offsets = {
    "up": (0, 20),
    "down": (0, -20),
    "left": (-20, 0),
    "right": (20, 0)
}

paused = False  # Variable to track whether the game is paused or not

# Create a turtle for displaying the score
score_display = turtle.Turtle()
score_display.speed(0)  # Set the drawing speed
score_display.color("white")  # Set the text color
score_display.penup()
score_display.hideturtle()
score_display.goto(0, h / 2 - 40)
score_display.write("Score: 0", align="center", font=("Courier", 24, "normal"))

# Define the game over screen
game_over_screen = turtle.Turtle()
game_over_screen.speed(0)
game_over_screen.color("white")
game_over_screen.penup()
game_over_screen.hideturtle()
game_over_screen.goto(0, 0)

# Constants for high score
high_score_file = "high_score.txt"
high_score = 0

# Function to read the high score from the file
def load_high_score():
    global high_score
    try:
        with open(high_score_file, "r") as file:
            high_score = int(file.read())
    except FileNotFoundError:
        high_score = 0

# Function to save the high score to the file
def save_high_score():
    with open(high_score_file, "w") as file:
        file.write(str(high_score))

# Function to update the high score if needed
def update_high_score():
    global high_score
    if score > high_score:
        high_score = score
        save_high_score()

# Function to display the high score on the screen
def display_high_score():
    score_display.clear()
    score_display.write(f"Score: {score}  High Score: {high_score}", align="center", font=("Courier", 24, "normal"))

def display_game_over():
    game_over_screen.clear()
    game_over_screen.write("Game Over", align="center", font=("Courier", 36, "normal"))
    game_over_screen.goto(0, -40)
    game_over_screen.write(f"Score: {score}", align="center", font=("Courier", 24, "normal"))
    game_over_screen.goto(0, -80)
    game_over_screen.write("Press 'R' to Restart", align="center", font=("Courier", 18, "normal"))

def game_over():
    global snake, snake_dir, food_position, pen, score, level, delay
    snake_dir = "stop"  # Stop snake movement
    update_high_score()  # Check and update the high score
    display_game_over()

def update_score():
    score_display.clear()
    score_display.write(f"Score: {score}", align="center", font=("Courier", 24, "normal"))

def reset():
    global snake, snake_dir, food_position, pen, score, level, delay, paused
    delay = initial_delay  # Reset delay
    snake = [[0, 0], [0, 20], [0, 40], [0, 60], [0, 80]]
    snake_dir = "up"
    food_position = get_random_food_position()
    food.goto(food_position)
    move_snake()
    score = 0
    level = 1
    load_high_score()
    update_score()  # Update the score display
    display_high_score()  # Display high score
    pen.clear()
    game_over_screen.clear()  # Clear the game over screen
    screen.update()
    screen.ontimer(display_high_score, 2000)  # Display high score for 2 seconds
    paused = False  # Reset the paused state

def move_snake():
    global snake, snake_dir, food_position, pen, score, level, delay, paused

    if paused:
        turtle.ontimer(move_snake, 100)
        return

    new_head = snake[-1].copy()
    new_head[0] = snake[-1][0] + offsets[snake_dir][0]
    new_head[1] = snake[-1][1] + offsets[snake_dir][1]

    # Check if the snake hits the left or right boundaries
    if new_head[0] < left_boundary or new_head[0] > right_boundary:
        game_over()  # Call the game over function
        return

    # Wrap around the top and bottom boundaries
    if new_head[1] > top_boundary:
        new_head[1] = bottom_boundary
    elif new_head[1] < bottom_boundary:
        new_head[1] = top_boundary

    if new_head in snake[:-1]:
        game_over()  # Call the game over function
        return

    snake.append(new_head)

    if not food_collision():
        snake.pop(0)

    pen.clearstamps()

    for segment in snake:
        pen.goto(segment[0], segment[1])
        pen.stamp()

    screen.update()

    if score % level_increment == 0:
        level += 1
        delay = max(delay - 10, min_delay)

    turtle.ontimer(move_snake, delay)

def food_collision():
    global food_position, score
    if get_distance(snake[-1], food_position) < 20:
        food_position = get_random_food_position()
        food.goto(food_position)
        score += 1
        update_score()  # Update the score display
        return True
    return False

def get_random_food_position():
    while True:
        x = random.randint(left_boundary, right_boundary)
        y = random.randint(bottom_boundary, top_boundary)
        if (x, y) not in snake:
            return (x, y)

def get_distance(pos1, pos2):
    x1, y1 = pos1
    x2, y2 = pos2
    distance = ((y2 - y1) ** 2 + (x2 - x1) ** 2) ** 0.5
    return distance

def go_up():
    global snake_dir
    if snake_dir != "down":
        snake_dir = "up"

def go_right():
    global snake_dir
    if snake_dir != "left":
        snake_dir = "right"

def go_down():
    global snake_dir
    if snake_dir != "up":
        snake_dir = "down"

def go_left():
    global snake_dir
    if snake_dir != "right":
        snake_dir = "left"

def toggle_pause():
    global paused
    paused = not paused
    if paused:
        score_display.clear()
        score_display.write("Game Paused", align="center", font=("Courier", 36, "normal"))
    else:
        score_display.clear()
        update_score()
        display_high_score()

# Create the screen
screen = turtle.Screen()
screen.setup(w, h)
screen.title("Snake")
screen.bgcolor("red")
screen.tracer(0)

# Create the pen and food
pen = turtle.Turtle("square")
pen.penup()

food = turtle.Turtle()
food.shape("circle")
food.color("blue")
food.shapesize(food_size / 20)
food.penup()

# Register the event handlers
screen.listen()
screen.onkey(go_up, "Up")
screen.onkey(go_right, "Right")
screen.onkey(go_down, "Down")
screen.onkey(go_left, "Left")
screen.onkey(reset, "r")  # Add a key to restart the game
screen.onkey(toggle_pause, "p")  # Add a key to pause/resume the game

reset()

# Start the game
turtle.done()
