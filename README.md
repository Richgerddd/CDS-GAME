import pygame
import random
import sys
from pygame import mixer

# Initialize Pygame
pygame.init()
mixer.init()

# Set up the display
WIDTH = 800
HEIGHT = 600
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Math Space Shooter")

# Colors
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
RED = (255, 0, 0)

# Player
class Player:
    def __init__(self):
        self.width = 50
        self.height = 50
        self.x = WIDTH // 2 - self.width // 2
        self.y = HEIGHT - 100
        self.speed = 5
        self.score = 0
        
    def draw(self):
        pygame.draw.rect(screen, WHITE, (self.x, self.y, self.width, self.height))

    def move(self):
        keys = pygame.key.get_pressed()
        if keys[pygame.K_LEFT] and self.x > 0:
            self.x -= self.speed
        if keys[pygame.K_RIGHT] and self.x < WIDTH - self.width:
            self.x += self.speed

# Alien
class Alien:
    def __init__(self):
        self.width = 50
        self.height = 50
        self.x = random.randint(0, WIDTH - self.width)
        self.y = 0
        self.speed = 2
        self.num1 = random.randint(1, 10)
        self.num2 = random.randint(1, 10)
        self.operator = random.choice(['+', '-', '*', '/'])
        if self.operator == '/':
            self.num1 = self.num2 * random.randint(1, 10)
        self.problem = f"{self.num1} {self.operator} {self.num2}"
        self.answer = self.calculate_answer()
        
    def calculate_answer(self):
        if self.operator == '+':
            return self.num1 + self.num2
        elif self.operator == '-':
            return self.num1 - self.num2
        elif self.operator == '*':
            return self.num1 * self.num2
        else:
            return self.num1 // self.num2
            
    def draw(self):
        pygame.draw.rect(screen, RED, (self.x, self.y, self.width, self.height))
        font = pygame.font.Font(None, 24)
        text = font.render(self.problem, True, WHITE)
        screen.blit(text, (self.x, self.y - 20))
        
    def move(self):
        self.y += self.speed

# Game setup
player = Player()
aliens = []
font = pygame.font.Font(None, 36)
clock = pygame.time.Clock()
spawn_timer = 0
answer = ""
game_over = False

# Game loop
while True:
    # Event handling
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()
        if event.type == pygame.KEYDOWN and not game_over:
            if event.key in (pygame.K_RETURN, pygame.K_KP_ENTER) and answer:
                try:
                    player_answer = float(answer)
                    for alien in aliens[:]:
                        if abs(player_answer - alien.answer) < 0.01:
                            aliens.remove(alien)
                            player.score += 10
                    answer = ""
                except ValueError:
                    answer = ""
            elif event.key == pygame.K_BACKSPACE:
                answer = answer[:-1]
            elif event.unicode.isnumeric() or event.unicode == '.':
                answer += event.unicode

    if not game_over:
        # Update game state
        screen.fill(BLACK)
        player.move()
        player.draw()

        # Spawn aliens
        spawn_timer += 1
        if spawn_timer >= 120:  # Spawn every 2 seconds
            aliens.append(Alien())
            spawn_timer = 0

        # Update and draw aliens
        for alien in aliens[:]:
            alien.move()
            alien.draw()
            if alien.y > HEIGHT:
                game_over = True
            
        # Draw score and current answer
        score_text = font.render(f"Score: {player.score}", True, WHITE)
        screen.blit(score_text, (10, 10))
        answer_text = font.render(f"Answer: {answer}", True, WHITE)
        screen.blit(answer_text, (10, 50))
    else:
        # Game over screen
        game_over_text = font.render("GAME OVER", True, WHITE)
        final_score_text = font.render(f"Final Score: {player.score}", True, WHITE)
        screen.blit(game_over_text, (WIDTH//2 - 100, HEIGHT//2 - 50))
        screen.blit(final_score_text, (WIDTH//2 - 100, HEIGHT//2 + 50))

    pygame.display.flip()
    clock.tick(60)
