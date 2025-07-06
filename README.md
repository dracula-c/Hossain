import pygame
import random
import os
from plyer import notification

pygame.init()

# Screen
WIDTH, HEIGHT = 480, 700
win = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Ultimate Car Game")

# Load assets
car_img = pygame.image.load("car.png")
car_img = pygame.transform.scale(car_img, (60, 100))
coin_img = pygame.image.load("coin.png")
coin_img = pygame.transform.scale(coin_img, (40, 40))
explosion_img = pygame.image.load("explosion.png")
explosion_img = pygame.transform.scale(explosion_img, (80, 80))

# Sound
coin_sound = pygame.mixer.Sound("coin.wav")
crash_sound = pygame.mixer.Sound("crash.wav")

# Fonts
font = pygame.font.SysFont("Arial", 30)

# Clock
clock = pygame.time.Clock()

# Variables
car_x = WIDTH // 2 - 30
car_y = HEIGHT - 120
car_speed = 6
fuel = 100
health = 100
score = 0
game_over = False

# Coin
coin_x = random.randint(50, WIDTH - 50)
coin_y = -50
coin_speed = 4

# Explosion
exploding = False
explosion_timer = 0

# Obstacles
obstacles = []
for i in range(3):
    x = random.randint(0, WIDTH - 60)
    y = random.randint(-600, -100)
    obstacles.append(pygame.Rect(x, y, 60, 100))

# Touch buttons
left_button = pygame.Rect(10, HEIGHT - 80, 80, 60)
right_button = pygame.Rect(WIDTH - 90, HEIGHT - 80, 80, 60)

# Save high score
high_score = 0
if os.path.exists("highscore.txt"):
    with open("highscore.txt", "r") as f:
        high_score = int(f.read())

# Helper
def draw_ui():
    pygame.draw.rect(win, (200, 200, 200), left_button)
    pygame.draw.rect(win, (200, 200, 200), right_button)
    win.blit(font.render("◀", True, (0, 0, 0)), (35, HEIGHT - 70))
    win.blit(font.render("▶", True, (0, 0, 0)), (WIDTH - 65, HEIGHT - 70))
    win.blit(font.render(f"Fuel: {fuel}", True, (255, 255, 255)), (10, 10))
    win.blit(font.render(f"Health: {health}", True, (255, 255, 255)), (10, 40))
    win.blit(font.render(f"Score: {score}", True, (255, 255, 0)), (10, 70))
    win.blit(font.render(f"High Score: {high_score}", True, (0, 255, 255)), (10, 100))

def game_loop():
    global car_x, fuel, health, score, game_over, coin_x, coin_y, exploding, explosion_timer, high_score

    run = True
    while run:
        clock.tick(60)
        win.fill((50, 50, 50))

        # Events
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                run = False

            # Touch (mouse) button
            if event.type == pygame.MOUSEBUTTONDOWN:
                if left_button.collidepoint(event.pos) and car_x > 0:
                    car_x -= car_speed * 5
                if right_button.collidepoint(event.pos) and car_x < WIDTH - 60:
                    car_x += car_speed * 5

        keys = pygame.key.get_pressed()
        if keys[pygame.K_LEFT] and car_x > 0:
            car_x -= car_speed
        if keys[pygame.K_RIGHT] and car_x < WIDTH - 60:
            car_x += car_speed

        # Move obstacles
        for obs in obstacles:
            obs.y += 6
            if obs.y > HEIGHT:
                obs.y = random.randint(-600, -100)
                obs.x = random.randint(0, WIDTH - 60)

            if pygame.Rect(car_x, car_y, 60, 100).colliderect(obs):
                crash_sound.play()
                health -= 20
                exploding = True
                explosion_timer = pygame.time.get_ticks()

        # Coin
        coin_y += coin_speed
        if coin_y > HEIGHT:
            coin_y = -50
            coin_x = random.randint(50, WIDTH - 50)
        if pygame.Rect(car_x, car_y, 60, 100).colliderect(pygame.Rect(coin_x, coin_y, 40, 40)):
            coin_sound.play()
            score += 10
            coin_y = -50
            coin_x = random.randint(50, WIDTH - 50)

        # Fuel consumption
        fuel -= 0.05
        if fuel <= 0 or health <= 0:
            game_over = True
            if score > high_score:
                high_score = score
                with open("highscore.txt", "w") as f:
                    f.write(str(high_score))
            notification.notify(title="Game Over", message=f"Score: {score}", timeout=3)
            pygame.time.delay(2000)
            return  # Restart loop

        # Drawing
        win.blit(car_img, (car_x, car_y))
        win.blit(coin_img, (coin_x, coin_y))

        for obs in obstacles:
            pygame.draw.rect(win, (255, 0, 0), obs)

        if exploding and pygame.time.get_ticks() - explosion_timer < 500:
            win.blit(explosion_img, (car_x - 10, car_y - 10))
        else:
            exploding = False

        draw_ui()
        pygame.display.update()

    pygame.quit()

game_loop()
