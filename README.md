# AI-MINI_PROJECT
"""
Tic-Tac-Toe AI using MiniMax with Alpha-Beta Pruning
Enhanced GUI version using Pygame with animations
"""

import pygame
import math
import time
import sys

# Initialize Pygame
pygame.init()

# Constants
WIDTH, HEIGHT = 800, 900
BOARD_SIZE = 600
LINE_WIDTH = 15
CIRCLE_RADIUS = 80
CIRCLE_WIDTH = 15
CROSS_WIDTH = 25
SPACE = 55

# Colors
BG_COLOR = (28, 170, 156)
LINE_COLOR = (23, 145, 135)
CIRCLE_COLOR = (239, 231, 200)
CROSS_COLOR = (66, 66, 66)
TEXT_COLOR = (255, 255, 255)
BUTTON_COLOR = (52, 152, 219)
BUTTON_HOVER = (41, 128, 185)

# Setup display
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption('Tic-Tac-Toe AI - MiniMax with Alpha-Beta Pruning')
clock = pygame.time.Clock()

# Fonts
title_font = pygame.font.Font(None, 48)
stats_font = pygame.font.Font(None, 28)
button_font = pygame.font.Font(None, 32)


class TicTacToeGame:
    """Tic-Tac-Toe game with AI using MiniMax"""
    
    def __init__(self):
        """Initialize game state"""
        self.board = [[' ' for _ in range(3)] for _ in range(3)]
        self.human = 'O'
        self.ai = 'X'
        self.game_over = False
        self.winner = None
        
        # Statistics
        self.nodes_evaluated = 0
        self.pruning_count = 0
        self.time_taken = 0
        
        # Score tracking
        self.games_played = 0
        self.human_wins = 0
        self.ai_wins = 0
        self.draws = 0
        
        # Animation
        self.ai_thinking = False
        self.thinking_dots = 0
        self.thinking_timer = 0
    
    def draw_lines(self):
        """Draw the game board lines"""
        # Vertical lines
        pygame.draw.line(screen, LINE_COLOR, (200, 150), (200, 750), LINE_WIDTH)
        pygame.draw.line(screen, LINE_COLOR, (400, 150), (400, 750), LINE_WIDTH)
        # Horizontal lines
        pygame.draw.line(screen, LINE_COLOR, (0, 350), (600, 350), LINE_WIDTH)
        pygame.draw.line(screen, LINE_COLOR, (0, 550), (600, 550), LINE_WIDTH)
    
    def draw_figures(self):
        """Draw X's and O's on the board"""
        for row in range(3):
            for col in range(3):
                if self.board[row][col] == 'O':
                    self.draw_circle(row, col)
                elif self.board[row][col] == 'X':
                    self.draw_cross(row, col)
    
    def draw_circle(self, row, col):
        """Draw O (circle) at specified position"""
        center_x = col * 200 + 100
        center_y = row * 200 + 250
        pygame.draw.circle(screen, CIRCLE_COLOR, (center_x, center_y), CIRCLE_RADIUS, CIRCLE_WIDTH)
    
    def draw_cross(self, row, col):
        """Draw X (cross) at specified position"""
        start_x = col * 200 + SPACE
        start_y = row * 200 + 150 + SPACE
        end_x = col * 200 + 200 - SPACE
        end_y = row * 200 + 350 - SPACE
        
        pygame.draw.line(screen, CROSS_COLOR, (start_x, start_y), (end_x, end_y), CROSS_WIDTH)
        pygame.draw.line(screen, CROSS_COLOR, (end_x, start_y), (start_x, end_y), CROSS_WIDTH)
    
    def evaluate(self):
        """Evaluate the board state"""
        # Check rows
        for row in self.board:
            if row[0] == row[1] == row[2] != ' ':
                return 10 if row[0] == self.ai else -10
        
        # Check columns
        for col in range(3):
            if self.board[0][col] == self.board[1][col] == self.board[2][col] != ' ':
                return 10 if self.board[0][col] == self.ai else -10
        
        # Check diagonals
        if self.board[0][0] == self.board[1][1] == self.board[2][2] != ' ':
            return 10 if self.board[0][0] == self.ai else -10
        
        if self.board[0][2] == self.board[1][1] == self.board[2][0] != ' ':
            return 10 if self.board[0][2] == self.ai else -10
        
        return 0
    
    def is_moves_left(self):
        """Check if there are moves left"""
        for row in self.board:
            if ' ' in row:
                return True
        return False
    
    def minimax(self, depth, is_max, alpha, beta):
        """MiniMax with Alpha-Beta pruning"""
        self.nodes_evaluated += 1
        
        score = self.evaluate()
        
        if score == 10:
            return score - depth
        if score == -10:
            return score + depth
        if not self.is_moves_left():
            return 0
        
        if is_max:
            best = -math.inf
            
            for i in range(3):
                for j in range(3):
                    if self.board[i][j] == ' ':
                        self.board[i][j] = self.ai
                        best = max(best, self.minimax(depth + 1, False, alpha, beta))
                        self.board[i][j] = ' '
                        
                        alpha = max(alpha, best)
                        if beta <= alpha:
                            self.pruning_count += 1
                            break
                if beta <= alpha:
                    break
            
            return best
        else:
            best = math.inf
            
            for i in range(3):
                for j in range(3):
                    if self.board[i][j] == ' ':
                        self.board[i][j] = self.human
                        best = min(best, self.minimax(depth + 1, True, alpha, beta))
                        self.board[i][j] = ' '
                        
                        beta = min(beta, best)
                        if beta <= alpha:
                            self.pruning_count += 1
                            break
                if beta <= alpha:
                    break
            
            return best
    
    def find_best_move(self):
        """Find the best move for AI"""
        self.nodes_evaluated = 0
        self.pruning_count = 0
        
        best_val = -math.inf
        best_move = (-1, -1)
        
        start_time = time.time()
        
        for i in range(3):
            for j in range(3):
                if self.board[i][j] == ' ':
                    self.board[i][j] = self.ai
                    move_val = self.minimax(0, False, -math.inf, math.inf)
                    self.board[i][j] = ' '
                    
                    if move_val > best_val:
                        best_move = (i, j)
                        best_val = move_val
        
        self.time_taken = (time.time() - start_time) * 1000
        
        return best_move
    
    def check_winner(self):
        """Check if game has ended"""
        score = self.evaluate()
        
        if score == 10:
            self.winner = 'AI'
            self.ai_wins += 1
            self.games_played += 1
            self.game_over = True
            return True
        elif score == -10:
            self.winner = 'Human'
            self.human_wins += 1
            self.games_played += 1
            self.game_over = True
            return True
        elif not self.is_moves_left():
            self.winner = 'Draw'
            self.draws += 1
            self.games_played += 1
            self.game_over = True
            return True
        
        return False
    
    def make_move(self, row, col):
        """Make a move at specified position"""
        if self.board[row][col] == ' ' and not self.game_over:
            self.board[row][col] = self.human
            return True
        return False
    
    def ai_move(self):
        """Let AI make a move"""
        self.ai_thinking = True
        move = self.find_best_move()
        if move != (-1, -1):
            self.board[move[0]][move[1]] = self.ai
        self.ai_thinking = False
    
    def reset(self):
        """Reset the game"""
        self.board = [[' ' for _ in range(3)] for _ in range(3)]
        self.game_over = False
        self.winner = None
        self.nodes_evaluated = 0
        self.pruning_count = 0
        self.time_taken = 0


class Button:
    """Button class for UI controls"""
    
    def __init__(self, x, y, width, height, text):
        self.rect = pygame.Rect(x, y, width, height)
        self.text = text
        self.color = BUTTON_COLOR
        self.hover = False
    
    def draw(self, surface):
        """Draw the button"""
        color = BUTTON_HOVER if self.hover else self.color
        pygame.draw.rect(surface, color, self.rect, border_radius=10)
        pygame.draw.rect(surface, TEXT_COLOR, self.rect, 3, border_radius=10)
        
        text_surf = button_font.render(self.text, True, TEXT_COLOR)
        text_rect = text_surf.get_rect(center=self.rect.center)
        surface.blit(text_surf, text_rect)
    
    def check_hover(self, pos):
        """Check if mouse is hovering over button"""
        self.hover = self.rect.collidepoint(pos)
    
    def is_clicked(self, pos):
        """Check if button is clicked"""
        return self.rect.collidepoint(pos)


def draw_ui(game, buttons):
    """Draw the complete UI"""
    screen.fill(BG_COLOR)
    
    # Title
    title_text = title_font.render('Tic-Tac-Toe AI', True, TEXT_COLOR)
    screen.blit(title_text, (WIDTH // 2 - title_text.get_width() // 2, 20))
    
    # Draw game board
    game.draw_lines()
    game.draw_figures()
    
    # Statistics
    if game.ai_thinking:
        dots = '.' * (game.thinking_dots + 1)
        stats_text = f"AI is thinking{dots}"
    else:
        stats_text = f"Nodes: {game.nodes_evaluated} | Pruned: {game.pruning_count} | Time: {game.time_taken:.2f}ms"
    
    stats_surface = stats_font.render(stats_text, True, TEXT_COLOR)
    screen.blit(stats_surface, (WIDTH // 2 - stats_surface.get_width() // 2, 770))
    
    # Score
    score_text = f"Games: {game.games_played} | You: {game.human_wins} | AI: {game.ai_wins} | Draws: {game.draws}"
    score_surface = stats_font.render(score_text, True, TEXT_COLOR)
    screen.blit(score_surface, (WIDTH // 2 - score_surface.get_width() // 2, 810))
    
    # Draw buttons
    for button in buttons:
        button.draw(screen)
    
    # Game over message
    if game.game_over:
        overlay = pygame.Surface((WIDTH, HEIGHT))
        overlay.set_alpha(180)
        overlay.fill((0, 0, 0))
        screen.blit(overlay, (0, 0))
        
        if game.winner == 'AI':
            result_text = "AI WINS! 🤖"
        elif game.winner == 'Human':
            result_text = "YOU WIN! 🎉"
        else:
            result_text = "IT'S A DRAW! 🤝"
        
        result_surface = title_font.render(result_text, True, TEXT_COLOR)
        screen.blit(result_surface, (WIDTH // 2 - result_surface.get_width() // 2, HEIGHT // 2 - 50))
        
        play_again_surface = stats_font.render('Click "New Game" to play again', True, TEXT_COLOR)
        screen.blit(play_again_surface, (WIDTH // 2 - play_again_surface.get_width() // 2, HEIGHT // 2 + 20))


def get_cell_from_mouse(pos):
    """Convert mouse position to board cell"""
    x, y = pos
    
    if y < 150 or y > 750 or x > 600:
        return None
    
    row = (y - 150) // 200
    col = x // 200
    
    return (row, col)


def main():
    """Main game loop"""
    game = TicTacToeGame()
    
    # Create buttons
    buttons = [
        Button(620, 200, 160, 50, 'New Game'),
        Button(620, 280, 160, 50, 'AI First'),
        Button(620, 360, 160, 50, 'Reset Stats')
    ]
    
    running = True
    
    while running:
        clock.tick(60)
        mouse_pos = pygame.mouse.get_pos()
        
        # Update thinking animation
        if game.ai_thinking:
            game.thinking_timer += 1
            if game.thinking_timer >= 30:
                game.thinking_dots = (game.thinking_dots + 1) % 3
                game.thinking_timer = 0
        
        # Event handling
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
            
            if event.type == pygame.MOUSEBUTTONDOWN:
                # Check button clicks
                if buttons[0].is_clicked(mouse_pos):  # New Game
                    game.reset()
                elif buttons[1].is_clicked(mouse_pos):  # AI First
                    game.reset()
                    pygame.time.wait(500)
                    game.ai_move()
                    game.check_winner()
                elif buttons[2].is_clicked(mouse_pos):  # Reset Stats
                    game.games_played = 0
                    game.human_wins = 0
                    game.ai_wins = 0
                    game.draws = 0
                    game.reset()
                
                # Check board clicks
                else:
                    cell = get_cell_from_mouse(mouse_pos)
                    if cell and not game.game_over and not game.ai_thinking:
                        row, col = cell
                        if game.make_move(row, col):
                            if not game.check_winner():
                                pygame.time.wait(300)
                                game.ai_move()
                                game.check_winner()
        
        # Update button hover states
        for button in buttons:
            button.check_hover(mouse_pos)
        
        # Draw everything
        draw_ui(game, buttons)
        pygame.display.flip()
    
    pygame.quit()
    sys.exit()


if __name__ == "__main__":
    main()
