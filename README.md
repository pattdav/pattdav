import pygame
import random

pygame.font.init()

# Constants
WIDTH, HEIGHT = 800, 700
PLAY_WIDTH, PLAY_HEIGHT = 300, 600
BLOCK_SIZE = 30

# Colors
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
RED = (255, 0, 0)
GREEN = (0, 255, 0)
BLUE = (0, 0, 255)
CYAN = (0, 255, 255)
YELLOW = (255, 255, 0)
PURPLE = (128, 0, 128)
ORANGE = (255, 165, 0)
GREY = (128, 128, 128)

# Shapes formats
S = [['.....',
      '.....',
      '..00.',
      '.00..',
      '.....'],
     ['.....',
      '..0..',
      '..00.',
      '...0.',
      '.....']]

Z = [['.....',
      '.....',
      '.00..',
      '..00.',
      '.....'],
     ['.....',
      '..0..',
      '.00..',
      '.0...',
      '.....']]

I = [['..0..',
      '..0..',
      '..0..',
      '..0..',
      '.....'],
     ['0000.',
      '.....',
      '.....',
      '.....',
      '.....']]

O = [['.....',
      '.....',
      '.00..',
      '.00..',
      '.....']]

J = [['.....',
      '.0...',
      '.000.',
      '.....',
      '.....'],
     ['.....',
      '..00.',
      '..0..',
      '..0..',
      '.....'],
     ['.....',
      '.....',
      '.000.',
      '...0.',
      '.....'],
     ['.....',
      '..0..',
      '..0..',
      '.00..',
      '.....']]

L = [['.....',
      '...0.',
      '.000.',
      '.....',
      '.....'],
     ['.....',
      '..0..',
      '..0..',
      '..00.',
      '.....'],
     ['.....',
      '.....',
      '.000.',
      '.0...',
      '.....'],
     ['.....',
      '.00..',
      '..0..',
      '..0..',
      '.....']]

T = [['.....',
      '..0..',
      '.000.',
      '.....',
      '.....'],
     ['.....',
      '..0..',
      '..00.',
      '..0..',
      '.....'],
     ['.....',
      '.....',
      '.000.',
      '..0..',
      '.....'],
     ['.....',
      '..0..',
      '.00..',
      '..0..',
      '.....']]

SHAPES = [S, Z, I, O, J, L, T]
SHAPE_COLORS = [RED, GREEN, CYAN, YELLOW, PURPLE, ORANGE, BLUE]


class Piece(object):
    rows = 20
    columns = 10

    def __init__(self, column, row, shape):
        self.x = column
        self.y = row
        self.shape = shape
        self.color = SHAPE_COLORS[SHAPES.index(shape)]
        self.rotation = 0

    def get_shape(self):
        return self.shape[self.rotation % len(self.shape)]

    def rotate(self):
        self.rotation += 1

    def draw(self, surface):
        shape_format = self.get_shape()
        for i, line in enumerate(shape_format):
            for j, column in enumerate(line):
                if column == '0':
                    pygame.draw.rect(surface, self.color, (self.x + j, self.y + i, 1, 1), 0)


def create_grid(locked_positions={}):
    grid = [[WHITE if (j, i) not in locked_positions else locked_positions[(j, i)] for j in range(10)] for i in range(20)]
    return grid


def convert_shape_format(piece):
    positions = []
    shape_format = piece.get_shape()
    for i, line in enumerate(shape_format):
        row = list(line)
        for j, column in enumerate(row):
            if column == '0':
                positions.append((piece.x + j, piece.y + i))
    for i, pos in enumerate(positions):
        positions[i] = (pos[0] // BLOCK_SIZE, pos[1] // BLOCK_SIZE)
    return positions


def valid_space(piece, grid):
    accepted_positions = [[(j, i) for j in range(10) if grid[i][j] == WHITE] for i in range(20)]
    accepted_positions = [j for sub in accepted_positions for j in sub]

    formatted = convert_shape_format(piece)
    for pos in formatted:
        if pos not in accepted_positions:
            if pos[1] > -1:
                return False
    return True


def check_lost(positions):
    for pos in positions:
        x, y = pos
        if y < 1:
            return True
    return False


def get_shape():
    return Piece(5, 0, random.choice(SHAPES))


def draw_grid(surface, row, col):
    sx = WIDTH // 2 - PLAY_WIDTH // 2
    sy = HEIGHT - PLAY_HEIGHT
    for i in range(row):
        pygame.draw.line(surface, GREY, (sx, sy + i * BLOCK_SIZE), (sx + PLAY_WIDTH, sy + i * BLOCK_SIZE))
        for j in range(col):
            pygame.draw.line(surface, GREY, (sx + j * BLOCK_SIZE, sy), (sx + j * BLOCK_SIZE, sy + PLAY_HEIGHT))


def clear_rows(grid, locked):
    inc = 0
    for i in range(len(grid)-1, -1, -1):
        row = grid[i]
        if WHITE not in row:
            inc += 1
            ind = i
            for j in range(len(row)):
                try:
                    del locked[(j, i)]
                except:
                    continue

    if inc > 0:
        for key in sorted(list(locked), key=lambda x: x[1])[::-1]:
            x, y = key
            if y < ind:
                newKey = (x, y + inc)
                locked[newKey] = locked.pop(key)

    return inc


def draw_next_shape(piece, surface):
    font = pygame.font.SysFont('comicsans', 30)
    label = font.render('Next Shape', 1, WHITE)

    sx = WIDTH // 2 - PLAY_WIDTH // 2 + PLAY_WIDTH + 50
    sy = HEIGHT // 2 - PLAY_HEIGHT // 2
    shape_format = piece.shape[piece.rotation % len(piece.shape)]

    for i, line in enumerate(shape_format):
        row = list(line)
        for j, column in enumerate(row):
            if column == '0':
                pygame.draw.rect(surface, piece.color, (sx + j*BLOCK_SIZE, sy + 
                
