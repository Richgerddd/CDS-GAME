# CDS-GAME

import pygame
import random

# Initialize Pygame
pygame.init()

# Constants
SCREEN_WIDTH = 800
SCREEN_HEIGHT = 600
PLAYER_WIDTH = 50
PLAYER_HEIGHT = 50
ENEMY_WIDTH = 50
ENEMY_HEIGHT = 50
BULLET_WIDTH = 5
BULLET_HEIGHT = 10
ENEMY_COUNT = 5

# Colors
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
RED = (255, 0, 0)
GREEN = (0, 255, 0)

# Set up the screen
screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
pygame.display.set_caption("Space Shooter")

# Player class
class Player:
    def __init__(self):
        self.rect = pygame.Rect(SCREEN_WIDTH // 2, SCREEN_HEIGHT - PLAYER_HEIGHT - 10, PLAYER_WIDTH, PLAYER_HEIGHT)

    def move(self, dx):
        self.rect.x += dx
        # Keep player within screen bounds
        if self.rect.x < 0:
            self.rect.x = 0
        if self.rect.x > SCREEN_WIDTH - PLAYER_WIDTH:
            self.rect.x = SCREEN_WIDTH - PLAYER_WIDTH

    def draw(self):
        pygame.draw.rect(screen, GREEN, self.rect)

# Bullet class
class Bullet:
    def __init__(self, x, y):
        self.rect = pygame.Rect(x, y, BULLET_WIDTH, BULLET_HEIGHT)

    def move(self):
        self.rect.y -= 5  # Move bullet upwards

    def draw(self):
        pygame.draw.rect(screen, WHITE, self.rect)

# Enemy class
class Enemy:
    def __init__(self):
        self.rect = pygame.Rect(random.randint(0, SCREEN_WIDTH - ENEMY_WIDTH), random.randint(-100, -40), ENEMY_WIDTH, ENEMY_HEIGHT)

    def move(self):
        self.rect.y += 3  # Move enemy downwards

    def draw(self):
        pygame.draw.rect(screen, RED, self.rect)

# Main game loop
def main():
    clock = pygame.time.Clock()
    player = Player()
    bullets = []
    enemies = [Enemy() for _ in range(ENEMY_COUNT)]
    running = True

    while running:
        screen.fill(BLACK)

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False

        keys = pygame.key.get_pressed()
        if keys[pygame.K_LEFT]:
            player.move(-5)
        if keys[pygame.K_RIGHT]:
            player.move(5)
        if keys[pygame.K_SPACE]:
            if len(bullets) < 5:  # Limit the number of bullets
                bullets.append(Bullet(player.rect.centerx - BULLET_WIDTH // 2, player.rect.top))

        # Move bullets
        for bullet in bullets[:]:
            bullet.move()
            if bullet.rect.y < 0:
                bullets.remove(bullet)

        # Move enemies
        for enemy in enemies[:]:
            enemy.move()
            if enemy.rect.y > SCREEN_HEIGHT:
                enemies.remove(enemy)
                enemies.append(Enemy())  # Respawn enemy

            # Check for bullet collision with enemy
            for bullet in bullets[:]:
                if bullet.rect.colliderect(enemy.rect):
                    bullets.remove(bullet)
                    enemies.remove(enemy)
                    enemies.append(Enemy())  # Respawn enemy
                    break

        # Draw everything
        player.draw()
        for bullet in bullets:
            bullet.draw()
        for enemy in enemies:
            enemy.draw()

        pygame.display.flip()
        clock.tick(60)

    pygame.quit()

if __name__ == "__main__":
    main()
