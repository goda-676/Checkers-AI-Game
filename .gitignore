#import pygame
import sys
import copy

# ==================== CONSTANTS ====================
WIDTH, HEIGHT = 800, 800
ROWS, COLS = 8, 8
SQUARE_SIZE = WIDTH // COLS

# Colors
RED = (255, 0, 0)
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
BLUE = (0, 0, 255)
GREY = (128, 128, 128)
GOLD = (255, 215, 0)
GREEN = (0, 255, 0)

# ==================== PIECE CLASS ====================
class Piece:
    PADDING = 15
    OUTLINE = 2

    def __init__(self, row, col, color):
        self.row = row
        self.col = col
        self.color = color
        self.king = False
        self.x = 0
        self.y = 0
        self.calc_pos()

    def calc_pos(self):
        self.x = SQUARE_SIZE * self.col + SQUARE_SIZE // 2
        self.y = SQUARE_SIZE * self.row + SQUARE_SIZE // 2

    def make_king(self):
        self.king = True

    def draw(self, win):
        radius = SQUARE_SIZE // 2 - self.PADDING
        pygame.draw.circle(win, GREY, (self.x, self.y), radius + self.OUTLINE)
        pygame.draw.circle(win, self.color, (self.x, self.y), radius)
        if self.king:
            # Draw crown using shapes instead of image
            crown_size = 20
            pygame.draw.circle(win, GOLD, (self.x, self.y), crown_size // 2)
            pygame.draw.polygon(win, GOLD, [
                (self.x - 10, self.y),
                (self.x - 5, self.y - 10),
                (self.x, self.y),
                (self.x + 5, self.y - 10),
                (self.x + 10, self.y)
            ])

    def move(self, row, col):
        self.row = row
        self.col = col
        self.calc_pos()

    def __repr__(self):
        return str(self.color)


# ==================== BOARD CLASS ====================
class Board:
    def __init__(self):
        self.board = []
        self.red_left = self.white_left = 12
        self.red_kings = self.white_kings = 0
        self.create_board()
    
    def draw_squares(self, win):
        win.fill(BLACK)
        for row in range(ROWS):
            for col in range(row % 2, COLS, 2):
                pygame.draw.rect(win, RED, (col * SQUARE_SIZE, row * SQUARE_SIZE, SQUARE_SIZE, SQUARE_SIZE))

    def move(self, piece, row, col):
        self.board[piece.row][piece.col], self.board[row][col] = self.board[row][col], self.board[piece.row][piece.col]
        piece.move(row, col)
        if row == ROWS - 1 or row == 0:
            piece.make_king()
            if piece.color == WHITE:
                self.white_kings += 1
            else:
                self.red_kings += 1 

    def get_piece(self, row, col):
        return self.board[row][col]

    def create_board(self):
        for row in range(ROWS):
            self.board.append([])
            for col in range(COLS):
                if col % 2 == ((row + 1) % 2):
                    if row < 3:
                        self.board[row].append(Piece(row, col, WHITE))
                    elif row > 4:
                        self.board[row].append(Piece(row, col, RED))
                    else:
                        self.board[row].append(0)
                else:
                    self.board[row].append(0)
        
    def draw(self, win):
        self.draw_squares(win)
        for row in range(ROWS):
            for col in range(COLS):
                piece = self.board[row][col]
                if piece != 0:
                    piece.draw(win)

    def remove(self, pieces):
        for piece in pieces:
            self.board[piece.row][piece.col] = 0
            if piece != 0:
                if piece.color == RED:
                    self.red_left -= 1
                else:
                    self.white_left -= 1
    
    def winner(self):
        if self.red_left <= 0:
            return WHITE
        elif self.white_left <= 0:
            return RED
        return None 

    def get_valid_moves(self, piece):
        moves = {}
        left = piece.col - 1
        right = piece.col + 1
        row = piece.row

        if piece.color == RED or piece.king:
            moves.update(self._traverse_left(row - 1, max(row - 3, -1), -1, piece.color, left))
            moves.update(self._traverse_right(row - 1, max(row - 3, -1), -1, piece.color, right))
        if piece.color == WHITE or piece.king:
            moves.update(self._traverse_left(row + 1, min(row + 3, ROWS), 1, piece.color, left))
            moves.update(self._traverse_right(row + 1, min(row + 3, ROWS), 1, piece.color, right))
        return moves

    def _traverse_left(self, start, stop, step, color, left, skipped=None):
        if skipped is None:
            skipped = []
        moves = {}
        last = []
        for r in range(start, stop, step):
            if left < 0:
                break
            current = self.board[r][left]
            if current == 0:
                if skipped and not last:
                    break
                elif skipped:
                    moves[(r, left)] = last + skipped
                else:
                    moves[(r, left)] = last
                if last:
                    if step == -1:
                        row = max(r - 3, 0)
                    else:
                        row = min(r + 3, ROWS)
                    moves.update(self._traverse_left(r + step, row, step, color, left - 1, skipped=last))
                    moves.update(self._traverse_right(r + step, row, step, color, left + 1, skipped=last))
                break
            elif current.color == color:
                break
            else:
                last = [current]
            left -= 1
        return moves

    def _traverse_right(self, start, stop, step, color, right, skipped=None):
        if skipped is None:
            skipped = []
        moves = {}
        last = []
        for r in range(start, stop, step):
            if right >= COLS:
                break
            current = self.board[r][right]
            if current == 0:
                if skipped and not last:
                    break
                elif skipped:
                    moves[(r, right)] = last + skipped
                else:
                    moves[(r, right)] = last
                if last:
                    if step == -1:
                        row = max(r - 3, 0)
                    else:
                        row = min(r + 3, ROWS)
                    moves.update(self._traverse_left(r + step, row, step, color, right - 1, skipped=last))
                    moves.update(self._traverse_right(r + step, row, step, color, right + 1, skipped=last))
                break
            elif current.color == color:
                break
            else:
                last = [current]
            right += 1
        return moves
    
    def get_all_pieces(self, color):
        """Get all pieces of a specific color"""
        pieces = []
        for row in self.board:
            for piece in row:
                if piece != 0 and piece.color == color:
                    pieces.append(piece)
        return pieces
    
    def evaluate(self):
        """
        Evaluation function for the board state
        Positive score favors WHITE, negative favors RED
        """
        # Basic piece count
        score = self.white_left - self.red_left
        
        # Kings are worth more (2.5x regular piece)
        score += (self.white_kings * 1.5) - (self.red_kings * 1.5)
        
        # Position evaluation: pieces closer to becoming king are better
        for row in range(ROWS):
            for col in range(COLS):
                piece = self.board[row][col]
                if piece != 0 and not piece.king:
                    if piece.color == WHITE:
                        # WHITE pieces moving down (higher row number = closer to king)
                        score += row * 0.1
                    else:
                        # RED pieces moving up (lower row number = closer to king)
                        score -= (ROWS - 1 - row) * 0.1
        
        return score


# ==================== AI CLASS WITH MINIMAX ====================
class AI:
    def __init__(self, depth=4):
        """
        Initialize AI with search depth
        depth=3: Easy
        depth=4: Medium (default)
        depth=5: Hard
        """
        self.depth = depth
        self.nodes_explored = 0
    
    def minimax(self, board, depth, alpha, beta, maximizing_player):
        """
        Minimax algorithm with Alpha-Beta Pruning
        
        Parameters:
        - board: Current board state
        - depth: How deep to search
        - alpha: Best value for maximizer
        - beta: Best value for minimizer
        - maximizing_player: True if WHITE's turn, False if RED's turn
        
        Returns: (score, best_board_state)
        """
        self.nodes_explored += 1
        
        # Base case: reached depth limit or game over
        if depth == 0 or board.winner() is not None:
            return board.evaluate(), board
        
        if maximizing_player:
            # WHITE's turn (maximizing)
            max_eval = float('-inf')
            best_board = None
            
            # Get all possible moves for WHITE
            all_moves = self.get_all_moves(board, WHITE)
            
            for move_board in all_moves:
                evaluation = self.minimax(move_board, depth - 1, alpha, beta, False)[0]
                
                if evaluation > max_eval:
                    max_eval = evaluation
                    best_board = move_board
                
                alpha = max(alpha, evaluation)
                
                # Alpha-Beta Pruning
                if beta <= alpha:
                    break
            
            return max_eval, best_board
        
        else:
            # RED's turn (minimizing)
            min_eval = float('inf')
            best_board = None
            
            # Get all possible moves for RED
            all_moves = self.get_all_moves(board, RED)
            
            for move_board in all_moves:
                evaluation = self.minimax(move_board, depth - 1, alpha, beta, True)[0]
                
                if evaluation < min_eval:
                    min_eval = evaluation
                    best_board = move_board
                
                beta = min(beta, evaluation)
                
                # Alpha-Beta Pruning
                if beta <= alpha:
                    break
            
            return min_eval, best_board
    
    def get_all_moves(self, board, color):
        """
        Generate all possible board states after one move for a color
        """
        moves = []
        
        # Get all pieces of this color
        for piece in board.get_all_pieces(color):
            # Get valid moves for this piece
            valid_moves = board.get_valid_moves(piece)
            
            # Simulate each move
            for move, skip in valid_moves.items():
                # Create a deep copy of the board
                temp_board = self.simulate_move(piece, move, board, skip)
                moves.append(temp_board)
        
        return moves
    
    def simulate_move(self, piece, move, board, skip):
        """
        Simulate a move and return new board state
        """
        # Deep copy the board
        temp_board = copy.deepcopy(board)
        temp_piece = temp_board.get_piece(piece.row, piece.col)
        new_row, new_col = move
        
        # Execute the move
        temp_board.move(temp_piece, new_row, new_col)
        
        # Remove skipped pieces
        if skip:
            temp_board.remove(skip)
        
        return temp_board
    
    def get_best_move(self, board, color):
        """
        Get the best move for AI using Minimax
        Returns the best board state
        """
        self.nodes_explored = 0
        
        # WHITE is maximizing, RED is minimizing
        is_maximizing = (color == WHITE)
        
        print(f"AI is thinking... (Depth: {self.depth})")
        _, best_board = self.minimax(board, self.depth, float('-inf'), float('inf'), is_maximizing)
        print(f"Nodes explored: {self.nodes_explored}")
        
        return best_board


# ==================== GAME CLASS ====================
class Game:
    def __init__(self, win, mode="PVP"):
        """
        mode: "PVP" (Player vs Player) or "PVE" (Player vs AI)
        """
        self.win = win
        self.mode = mode
        self.ai = AI(depth=4) if mode == "PVE" else None
        self._init()
    
    def _init(self):
        self.selected = None
        self.board = Board()
        self.turn = RED
        self.valid_moves = {}

    def update(self):
        self.board.draw(self.win)
        self.draw_valid_moves(self.valid_moves)
        self.draw_turn_indicator()
        pygame.display.update()

    def draw_turn_indicator(self):
        font = pygame.font.Font(None, 36)
        color_name = "RED (You)" if self.turn == RED else "WHITE (AI)" if self.mode == "PVE" else "WHITE"
        text = font.render(f"Turn: {color_name}", True, self.turn)
        self.win.blit(text, (10, 10))
        
        # Draw mode indicator
        mode_text = "Player vs AI" if self.mode == "PVE" else "Player vs Player"
        mode_surface = font.render(mode_text, True, GREEN)
        self.win.blit(mode_surface, (WIDTH - 250, 10))

    def winner(self):
        return self.board.winner()

    def reset(self):
        self._init()

    def select(self, row, col):
        if self.selected:
            result = self._move(row, col)
            if not result:
                self.selected = None
                self.select(row, col)
        piece = self.board.get_piece(row, col)
        if piece != 0 and piece.color == self.turn:
            self.selected = piece
            self.valid_moves = self.board.get_valid_moves(piece)
            return True
        return False

    def _move(self, row, col):
        piece = self.board.get_piece(row, col)
        if self.selected and piece == 0 and (row, col) in self.valid_moves:
            self.board.move(self.selected, row, col)
            skipped = self.valid_moves[(row, col)]
            if skipped:
                self.board.remove(skipped)
            self.change_turn()
            return True
        return False

    def draw_valid_moves(self, moves):
        for move in moves:
            row, col = move
            pygame.draw.circle(self.win, BLUE, 
                             (col * SQUARE_SIZE + SQUARE_SIZE // 2, 
                              row * SQUARE_SIZE + SQUARE_SIZE // 2), 15)

    def change_turn(self):
        self.valid_moves = {}
        self.turn = WHITE if self.turn == RED else RED
        self.selected = None
    
    def ai_move(self):
        """Execute AI move"""
        if self.mode == "PVE" and self.turn == WHITE and self.ai:
            # Get best move from AI
            new_board = self.ai.get_best_move(self.board, WHITE)
            self.board = new_board
            self.change_turn()


# ==================== MENU SCREEN ====================
def draw_menu(win):
    """Draw game mode selection menu"""
    win.fill(BLACK)
    
    font_title = pygame.font.Font(None, 72)
    font_option = pygame.font.Font(None, 48)
    
    # Title
    title = font_title.render("CHECKERS", True, WHITE)
    title_rect = title.get_rect(center=(WIDTH // 2, 150))
    win.blit(title, title_rect)
    
    # Options
    pvp_text = font_option.render("1. Player vs Player", True, RED)
    pvp_rect = pvp_text.get_rect(center=(WIDTH // 2, 300))
    
    pve_text = font_option.render("2. Player vs AI", True, GREEN)
    pve_rect = pve_text.get_rect(center=(WIDTH // 2, 400))
    
    instruction = font_option.render("Press 1 or 2 to select", True, GREY)
    instruction_rect = instruction.get_rect(center=(WIDTH // 2, 550))
    
    win.blit(pvp_text, pvp_rect)
    win.blit(pve_text, pve_rect)
    win.blit(instruction, instruction_rect)
    
    pygame.display.update()


# ==================== MAIN GAME ====================
def get_row_col_from_mouse(pos):
    x, y = pos
    row = y // SQUARE_SIZE
    col = x // SQUARE_SIZE
    return row, col


def main():
    pygame.init()
    FPS = 60
    WIN = pygame.display.set_mode((WIDTH, HEIGHT))
    pygame.display.set_caption('Checkers Game - AI Project')
    
    # Menu selection
    selecting_mode = True
    game_mode = None
    
    while selecting_mode:
        draw_menu(WIN)
        
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_1:
                    game_mode = "PVP"
                    selecting_mode = False
                elif event.key == pygame.K_2:
                    game_mode = "PVE"
                    selecting_mode = False
    
    # Start game with selected mode
    run = True
    clock = pygame.time.Clock()
    game = Game(WIN, mode=game_mode)

    while run:
        clock.tick(FPS)
        
        # Check for AI turn
        if game.mode == "PVE" and game.turn == WHITE:
            game.ai_move()
        
        winner = game.winner()
        if winner is not None:
            # Draw final state
            game.update()
            
            # Show winner message
            font = pygame.font.Font(None, 72)
            if game.mode == "PVE":
                winner_text = "YOU WIN!" if winner == RED else "AI WINS!"
            else:
                winner_text = "RED WINS!" if winner == RED else "WHITE WINS!"
            
            text = font.render(winner_text, True, winner)
            text_rect = text.get_rect(center=(WIDTH // 2, HEIGHT // 2))
            
            # Draw semi-transparent background
            overlay = pygame.Surface((WIDTH, HEIGHT))
            overlay.set_alpha(200)
            overlay.fill(BLACK)
            WIN.blit(overlay, (0, 0))
            WIN.blit(text, text_rect)
            
            pygame.display.update()
            pygame.time.delay(5000)
            run = False

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                run = False
            
            if event.type == pygame.MOUSEBUTTONDOWN:
                # Only allow human input on their turn
                if game.mode == "PVP" or (game.mode == "PVE" and game.turn == RED):
                    pos = pygame.mouse.get_pos()
                    row, col = get_row_col_from_mouse(pos)
                    game.select(row, col)

        game.update()
    
    pygame.quit()
    sys.exit()


if __name__ == "__main__":
    main()
