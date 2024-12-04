# Game
import pygame
import RPi.GPIO as GPIO
import random
import time

# GPIO setup
GPIO.setmode(GPIO.BCM)
GPIO.setup(17, GPIO.IN)  # Joystick X-axis
GPIO.setup(27, GPIO.IN)  # Joystick Y-axis
GPIO.setup(22, GPIO.IN, pull_up_down=GPIO.PUD_UP)  # Joystick button

# Pygame setup
pygame.init()

# Screen dimensions
WIDTH, HEIGHT = 800, 600
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Joystick Dodge Game")

# Colors
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
RED = (255, 0, 0)
BLUE = (0, 0, 255)

# Clock
clock = pygame.time.Clock()

# Player properties
player_size = 50
player_x = WIDTH // 2
player_y = HEIGHT - player_size - 10
player_speed = 10

# Obstacle properties
obstacle_width = 50
obstacle_height = 50
obstacle_speed = 10
obstacle_list = []

# Functions
def create_obstacle():
    x_pos = random.randint(0, WIDTH - obstacle_width)
    return [x_pos, 0]

def draw_obstacles(obstacles):
    for obstacle in obstacles:
        pygame.draw.rect(screen, RED, (obstacle[0], obstacle[1], obstacle_width, obstacle_height))

def move_obstacles(obstacles):
    for obstacle in obstacles:
        obstacle[1] += obstacle_speed

def check_collision(player_x, player_y, obstacles):
    for obstacle in obstacles:
        if (
            obstacle[0] < player_x < obstacle[0] + obstacle_width or
            obstacle[0] < player_x + player_size < obstacle[0] + obstacle_width
        ):
            if (
                obstacle[1] < player_y < obstacle[1] + obstacle_height or
                obstacle[1] < player_y + player_size < obstacle[1] + obstacle_height
            ):
                return True
    return False

def game_over_screen():
    screen.fill(BLACK)
    font = pygame.font.SysFont(None, 75)
    game_over_text = font.render("GAME OVER!", True, RED)
    screen.blit(game_over_text, (WIDTH // 4, HEIGHT // 3))
    pygame.display.flip()
    time.sleep(3)

# Main game loop
def game_loop():
    global player_x, player_y

    game_over = False
    score = 0

    # Obstacle generation
    obstacle_frequency = 30  # Frames per obstacle
    frame_count = 0

    while not game_over:
        # Event handling
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                game_over = True

        # Joystick control
        if GPIO.input(17) == 0:  # Joystick left
            player_x -= player_speed
        elif GPIO.input(17) == 1:  # Joystick right
            player_x += player_speed
        if GPIO.input(27) == 0:  # Joystick up
            player_y -= player_speed
        elif GPIO.input(27) == 1:  # Joystick down
            player_y += player_speed

        # Boundary conditions
        player_x = max(0, min(WIDTH - player_size, player_x))
        player_y = max(0, min(HEIGHT - player_size, player_y))

        # Obstacle handling
        if frame_count % obstacle_frequency == 0:
            obstacle_list.append(create_obstacle())

        move_obstacles(obstacle_list)
        obstacle_list[:] = [obstacle for obstacle in obstacle_list if obstacle[1] < HEIGHT]

        # Check collisions
        if check_collision(player_x, player_y, obstacle_list):
            game_over_screen()
            game_over = True
            continue

        # Draw everything
        screen.fill(BLACK)
        pygame.draw.rect(screen, BLUE, (player_x, player_y, player_size, player_size))
        draw_obstacles(obstacle_list)

        # Update score
        score += 1
        font = pygame.font.SysFont(None, 35)
        score_text = font.render(f"Score: {score}", True, WHITE)
        screen.blit(score_text, (10, 10))

        pygame.display.flip()
        clock.tick(30)
        frame_count += 1

    GPIO.cleanup()

# Start the game
game_loop()
pygame.quit()
