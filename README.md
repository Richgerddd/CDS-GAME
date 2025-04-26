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
GREEN = (0, 255, 0)
BLUE = (0, 0, 255)

# Button class
class Button:
    def __init__(self, x, y, width, height, text, color):
        self.rect = pygame.Rect(x, y, width, height)
        self.text = text
        self.color = color
        self.font = pygame.font.Font(None, 36)
        
    def draw(self):
        pygame.draw.rect(screen, self.color, self.rect)
        text_surface = self.font.render(self.text, True, BLACK)
        text_rect = text_surface.get_rect(center=self.rect.center)
        screen.blit(text_surface, text_rect)
        
    def is_clicked(self, pos):
        return self.rect.collidepoint(pos)

# Settings class
class Settings:
    def __init__(self):
        self.difficulty = "Normal"  # Easy, Normal, Hard
        self.alien_speed = 2
        self.spawn_rate = 120
        self.sound_on = True
        
    def adjust_difficulty(self):
        if self.difficulty == "Easy":
            self.alien_speed = 1
            self.spawn_rate = 180
        elif self.difficulty == "Normal":
            self.alien_speed = 2
            self.spawn_rate = 120
        else:  # Hard
            self.alien_speed = 3
            self.spawn_rate = 60

# Player class (same as before)
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

# Alien class (modified to use settings)
class Alien:
    def __init__(self, settings):
        self.width = 50
        self.height = 50
        self.x = random.randint(0, WIDTH - self.width)
        self.y = 0
        self.speed = settings.alien_speed
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

# Game class
class Game:
    def __init__(self):
        self.state = "menu"  # menu, playing, settings, game_over
        self.player = Player()
        self.aliens = []
        self.settings = Settings()
        self.spawn_timer = 0
        self.answer = ""
        self.setup_buttons()
        
    def setup_buttons(self):
        self.start_button = Button(WIDTH//2 - 100, HEIGHT//2 - 50, 200, 50, "Start Game", GREEN)
        self.settings_button = Button(WIDTH//2 - 100, HEIGHT//2 + 50, 200, 50, "Settings", BLUE)
        self.restart_button = Button(WIDTH//2 - 100, HEIGHT//2 + 100, 200, 50, "Restart", GREEN)
        self.back_button = Button(WIDTH//2 - 100, HEIGHT - 100, 200, 50, "Back", RED)
        self.difficulty_button = Button(WIDTH//2 - 100, HEIGHT//2 - 50, 200, 50, 
                                     f"Difficulty: {self.settings.difficulty}", BLUE)
        self.sound_button = Button(WIDTH//2 - 100, HEIGHT//2 + 50, 200, 50,
                                 f"Sound: {'On' if self.settings.sound_on else 'Off'}", BLUE)
        
    def handle_menu_click(self, pos):
        if self.start_button.is_clicked(pos):
            self.state = "playing"
        elif self.settings_button.is_clicked(pos):
            self.state = "settings"
            
    def handle_settings_click(self, pos):
        if self.difficulty_button.is_clicked(pos):
            if self.settings.difficulty == "Easy":
                self.settings.difficulty = "Normal"
            elif self.settings.difficulty == "Normal":
                self.settings.difficulty = "Hard"
            else:
                self.settings.difficulty = "Easy"
            self.settings.adjust_difficulty()
            self.difficulty_button.text = f"Difficulty: {self.settings.difficulty}"
        elif self.sound_button.is_clicked(pos):
            self.settings.sound_on = not self.settings.sound_on
            self.sound_button.text = f"Sound: {'On' if self.settings.sound_on else 'Off'}"
        elif self.back_button.is_clicked(pos):
            self.state = "menu"
            
    def handle_game_over_click(self, pos):
        if self.restart_button.is_clicked(pos):
            self.__init__()
            self.state = "playing"
        elif self.back_button.is_clicked(pos):
            self.__init__()
            
    def run(self):
        clock = pygame.time.Clock()
        
        while True:
            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    pygame.quit()
                    sys.exit()
                if event.type == pygame.MOUSEBUTTONDOWN:
                    if self.state == "menu":
                        self.handle_menu_click(event.pos)
                    elif self.state == "settings":
                        self.handle_settings_click(event.pos)
                    elif self.state == "game_over":
                        self.handle_game_over_click(event.pos)
                if self.state == "playing" and event.type == pygame.KEYDOWN:
                    if event.key in (pygame.K_RETURN, pygame.K_KP_ENTER) and self.answer:
                        try:
                            player_answer = float(self.answer)
                            for alien in self.aliens[:]:
                                if abs(player_answer - alien.answer) < 0.01:
                                    self.aliens.remove(alien)
                                    self.player.score += 10
                            self.answer = ""
                        except ValueError:
                            self.answer = ""
                    elif event.key == pygame.K_BACKSPACE:
                        self.answer = self.answer[:-1]
                    elif event.unicode.isnumeric() or event.unicode == '.':
                        self.answer += event.unicode

            screen.fill(BLACK)

            if self.state == "menu":
                self.start_button.draw()
                self.settings_button.draw()
                
            elif self.state == "settings":
                self.difficulty_button.draw()
                self.sound_button.draw()
                self.back_button.draw()
                
            elif self.state == "playing":
                self.player.move()
                self.player.draw()
                
                self.spawn_timer += 1
                if self.spawn_timer >= self.settings.spawn_rate:
                    self.aliens.append(Alien(self.settings))
                    self.spawn_timer = 0

                for alien in self.aliens[:]:
                    alien.move()
                    alien.draw()
                    if alien.y > HEIGHT:
                        self.state = "game_over"
                        
                score_text = pygame.font.Font(None, 36).render(f"Score: {self.player.score}", True, WHITE)
                screen.blit(score_text, (10, 10))
                answer_text = pygame.font.Font(None, 36).render(f"Answer: {self.answer}", True, WHITE)
                screen.blit(answer_text, (10, 50))
                
            elif self.state == "game_over":
                game_over_text = pygame.font.Font(None, 48).render("GAME OVER", True, WHITE)
                score_text = pygame.font.Font(None, 36).render(f"Final Score: {self.player.score}", True, WHITE)
                screen.blit(game_over_text, (WIDTH//2 - 100, HEIGHT//2 - 100))
                screen.blit(score_text, (WIDTH//2 - 100, HEIGHT//2))
                self.restart_button.draw()
                self.back_button.draw()

            pygame.display.flip()
            clock.tick(60)

# Start the game
if __name__ == "__main__":
    game = Game()
    game.run()

