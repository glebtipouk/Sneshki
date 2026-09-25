import sys
import math
import random
from PyQt6.QtWidgets import QApplication, QMainWindow, QWidget
from PyQt6.QtGui import QPainter, QColor, QPen, QBrush, QFont, QPainterPath
from PyQt6.QtCore import Qt, QTimer, QPointF, QRectF

# Константы
LOGICAL_WIDTH = 1200
LOGICAL_HEIGHT = 600
GROUND_Y = 500
GRAVITY = 0.45
FPS = 60

class GameState:
    MENU = "MENU"
    LEVEL_MODE = "LEVEL_MODE"
    TRAINING_MODE = "TRAINING_MODE"
    GAME_OVER = "GAME_OVER"
    VICTORY = "VICTORY"

class Snowball:
    def __init__(self, x, y, angle, power):
        self.x = x
        self.y = y
        self.radius = 8
        self.vx = math.cos(math.radians(angle)) * power
        self.vy = -math.sin(math.radians(angle)) * power
        self.active = True

    def update(self):
        self.x += self.vx
        self.y += self.vy
        self.vy += GRAVITY
        if self.y > GROUND_Y or self.x > LOGICAL_WIDTH or self.x < 0:
            self.active = False

class Obstacle:
    def __init__(self, x, w, h):
        self.x = x
        self.w = w
        self.h = h
        self.cx = x + w / 2  
        self.cy = GROUND_Y   

class Target:
    def __init__(self, x=None, hp=1, scale=1.0, is_boss=False):
        self.hp = hp
        self.max_hp = hp
        self.scale = scale
        self.is_boss = is_boss
        self.x = x if x else random.randint(450, 1100)
        self.y = GROUND_Y

    def reset(self, hp=1):
        self.x = random.randint(450, 1100)
        self.hp = hp
        self.max_hp = hp

class GameWidget(QWidget):
    def __init__(self):
        super().__init__()
        self.setMouseTracking(True)
        self.setFocusPolicy(Qt.FocusPolicy.StrongFocus)
        
        self.state = GameState.MENU
        self.showing_rules = False  
        
        self.game_timer = QTimer()
        self.game_timer.timeout.connect(self.update_game)
        self.game_timer.start(1000 // FPS)
        
        self.clock_timer = QTimer()
        self.clock_timer.timeout.connect(self.tick_seconds)
        self.clock_timer.start(1000)

        self.init_variables()

    def init_variables(self):
        self.score = 0
        self.level = 1
        self.snowballs_left = 15
        self.time_left = -1
        self.player_x = 100
        self.snowballs = []
        self.targets = []
        self.obstacles = []
        self.angle = 45
        self.power = 0
        self.power_phase = 0
        self.is_charging = False

    def start_level(self, level):
        self.state = GameState.LEVEL_MODE
        self.showing_rules = False
        self.level = level
        self.snowballs = []
        self.obstacles = []
        self.time_left = -1
        
        if level == 1:
            self.snowballs_left = 12
            self.targets = [Target(600, 1), Target(900, 1)]
            self.obstacles = [Obstacle(300, 100, 80)]
        elif level == 2:
            self.snowballs_left = 15
            self.targets = [Target(550, 1), Target(800, 2), Target(1000, 1)]
            self.obstacles = [Obstacle(300, 80, 100)]
        elif level == 3:
            self.snowballs_left = 15
            self.time_left = 40
            self.targets = [Target(550, 2), Target(750, 2), Target(950, 2)]
            self.obstacles = [Obstacle(280, 70, 120), Obstacle(650, 70, 80)]
        elif level == 4:
            self.snowballs_left = 20
            self.time_left = 45
            self.targets = [Target(500, 1), Target(700, 2), Target(900, 2), Target(1100, 3)]
            self.obstacles = [Obstacle(300, 120, 120)]
        elif level == 5:
            self.snowballs_left = 15
            self.time_left = 35
            self.targets = [Target(600, 3), Target(800, 3), Target(1000, 3)]
            self.obstacles = [Obstacle(350, 100, 140), Obstacle(700, 60, 100)]
        elif level == 6:
            self.snowballs_left = 25
            self.time_left = 40
            self.targets = [Target(450, 2), Target(650, 3), Target(850, 3), Target(1050, 4)]
            self.obstacles = [Obstacle(250, 80, 150), Obstacle(550, 60, 100)]
        elif level == 7:
            self.snowballs_left = 15
            self.time_left = 35
            self.targets = [Target(700, 4), Target(900, 4), Target(1100, 3)]
            self.obstacles = [Obstacle(300, 250, 130)]
        elif level == 8:
            self.snowballs_left = 25
            self.time_left = 40
            self.targets = [Target(500, 2), Target(700, 4), Target(900, 5), Target(1100, 4)]
            self.obstacles = [Obstacle(280, 70, 160), Obstacle(600, 70, 160)]
        elif level == 9:
            self.snowballs_left = 20
            self.time_left = 30
            self.targets = [Target(650, 4), Target(850, 5), Target(1050, 5)]
            self.obstacles = [Obstacle(300, 100, 150), Obstacle(550, 60, 130), Obstacle(750, 60, 130)]
        elif level == 10:
            self.snowballs_left = 30
            self.time_left = 60
            self.targets = [Target(950, hp=15, scale=0.5, is_boss=True)] 
            self.targets.extend([Target(800, hp=1, scale=0.8), Target(900, hp=1, scale=0.8), Target(1050, hp=1, scale=0.8)])
            self.obstacles = [Obstacle(350, 120, 130), Obstacle(650, 80, 100)]

    def start_training(self):
        self.init_variables()
        self.state = GameState.TRAINING_MODE
        self.showing_rules = False
        self.snowballs_left = 9999
        self.targets = [Target(scale=random.uniform(0.7, 1.2)) for _ in range(3)]
        self.obstacles = [Obstacle(300, 120, 100)]

    def tick_seconds(self):
        if self.state == GameState.LEVEL_MODE and self.time_left > 0:
            self.time_left -= 1
            if self.time_left <= 0: self.state = GameState.GAME_OVER

    def update_game(self):
        if self.state not in [GameState.LEVEL_MODE, GameState.TRAINING_MODE]:
            self.update(); return

        if self.is_charging:
            self.power_phase += 0.05
            self.power = 5 + 18 * abs(math.sin(self.power_phase))

        for sb in self.snowballs[:]:
            sb.update()
            
            for obs in self.obstacles:
                a = obs.w / 2 + sb.radius
                b = obs.h + sb.radius
                nx = (sb.x - obs.cx) / a
                ny = (sb.y - obs.cy) / b
                if nx*nx + ny*ny <= 1 and sb.y <= GROUND_Y + sb.radius:
                    sb.active = False
                    break

            if not sb.active:
                if sb in self.snowballs: self.snowballs.remove(sb)
                continue
                
            for target in self.targets[:]:
                hit = False
                for offset, r in [(-15, 25), (-45, 20), (-65, 15)]:
                    cy = target.y + (offset * target.scale)
                    r_scaled = r * target.scale
                    if math.hypot(sb.x - target.x, sb.y - cy) < sb.radius + r_scaled:
                        hit = True
                        break
                
                if hit:
                    target.hp -= 1
                    self.score += 100 * (10 if target.is_boss else 1)
                    if sb in self.snowballs: self.snowballs.remove(sb)
                    if target.hp <= 0:
                        if self.state == GameState.TRAINING_MODE:
                            target.reset(hp=1)
                        else:
                            self.targets.remove(target)
                    break

        if self.state == GameState.LEVEL_MODE:
            if not self.targets:
                if self.level < 10: self.start_level(self.level + 1)
                else: self.state = GameState.VICTORY
            
            if self.snowballs_left <= 0 and not self.snowballs and self.targets:
                self.state = GameState.GAME_OVER

        self.update()

    def get_scaling(self):
        scale = min(self.width() / LOGICAL_WIDTH, self.height() / LOGICAL_HEIGHT)
        return scale, (self.width() - LOGICAL_WIDTH * scale) / 2, (self.height() - LOGICAL_HEIGHT * scale) / 2

    def paintEvent(self, event):
        painter = QPainter(self)
        painter.setRenderHint(QPainter.RenderHint.Antialiasing)
        scale, off_x, off_y = self.get_scaling()
        painter.fillRect(self.rect(), QColor("#1a2a3a"))
        painter.translate(off_x, off_y)
        painter.scale(scale, scale)

        if self.state == GameState.MENU: self.draw_menu(painter)
        elif self.state in [GameState.LEVEL_MODE, GameState.TRAINING_MODE]: self.draw_gameplay(painter)
        elif self.state in [GameState.GAME_OVER, GameState.VICTORY]: self.draw_final_screen(painter)

    def draw_gameplay(self, painter):
        painter.fillRect(0, 0, LOGICAL_WIDTH, GROUND_Y, QColor("#DFF9FB"))
        painter.fillRect(0, GROUND_Y, LOGICAL_WIDTH, LOGICAL_HEIGHT - GROUND_Y, QColor("#FFFFFF"))
        
        for obs in self.obstacles:
            painter.setBrush(QColor("#FFFFFF"))
            painter.setPen(QPen(QColor("#B2BEC3"), 2))
            rect = QRectF(obs.x, GROUND_Y - obs.h, obs.w, obs.h * 2)
            painter.drawChord(rect, 0, 180 * 16) 
            painter.setPen(QPen(QColor("#DFF9FB"), 2))
            painter.drawLine(int(obs.cx - obs.w*0.2), int(GROUND_Y - obs.h*0.5), int(obs.cx + obs.w*0.2), int(GROUND_Y - obs.h*0.5))

        self.draw_player(painter)
        
        for t in self.targets:
            self.draw_snowman_target(painter, t)

        for sb in self.snowballs:
            painter.setBrush(Qt.GlobalColor.white)
            painter.setPen(QPen(QColor("#B2BEC3"), 1))
            painter.drawEllipse(QPointF(sb.x, sb.y), sb.radius, sb.radius)
            
        self.draw_ui(painter)

    def draw_player(self, painter):
        px, py = self.player_x, GROUND_Y
        painter.setBrush(QColor("#2980b9"))
        painter.drawRect(px, py - 40, 30, 40)
        painter.setBrush(QColor("#ffdbac"))
        painter.drawEllipse(QPointF(px + 15, py - 50), 12, 12)
        painter.setBrush(Qt.GlobalColor.white)
        painter.drawChord(int(px+5), int(py-55), 20, 20, 0, -180*16)
        painter.setBrush(QColor("#e74c3c"))
        pts = [QPointF(px+3, py-58), QPointF(px+27, py-58), QPointF(px+15, py-80)]
        painter.drawPolygon(pts)
        
        angle_rad = math.radians(self.angle)
        line_len = 40 + self.power * 2
        painter.setPen(QPen(Qt.GlobalColor.red, 2, Qt.PenStyle.DashLine))
        painter.drawLine(QPointF(px+15, py-45), 
                         QPointF(px+15 + math.cos(angle_rad)*line_len, 
                                 py-45 - math.sin(angle_rad)*line_len))

    def draw_snowman_target(self, painter, t):
        painter.save()
        painter.translate(t.x, t.y)
        painter.scale(t.scale, t.scale)
        
        if t.is_boss: body_color = QColor("#bdc3c7")
        elif t.max_hp > 1: body_color = QColor("#ecf0f1")
        else: body_color = Qt.GlobalColor.white

        painter.setBrush(body_color)
        painter.setPen(QPen(QColor("#95a5a6"), 1))
        painter.drawEllipse(QPointF(0, -15), 25, 25)
        painter.drawEllipse(QPointF(0, -45), 20, 20)
        painter.drawEllipse(QPointF(0, -65), 15, 15)
        
        painter.setBrush(QColor("#e67e22"))
        painter.drawPolygon([QPointF(0, -65), QPointF(15, -63), QPointF(0, -61)])
        painter.setBrush(Qt.GlobalColor.black)
        painter.drawEllipse(QPointF(-5, -68), 2, 2)
        painter.drawEllipse(QPointF(5, -68), 2, 2)
        
        if t.is_boss:
            painter.setBrush(QColor("#f1c40f")) 
            painter.drawPolygon([QPointF(-12, -75), QPointF(-15, -95), QPointF(-5, -85), 
                                 QPointF(0, -100), QPointF(5, -85), QPointF(15, -95), QPointF(12, -75)])
        else:
            painter.setBrush(QColor("#7f8c8d") if t.max_hp == 1 else QColor("#2c3e50"))
            painter.drawRect(-12, -85, 24, 10)
        
        painter.restore()

        if t.max_hp > 1 or t.is_boss:
            bar_w = 60 if t.is_boss else 45
            bar_h = 12
            bar_x = t.x - bar_w / 2
            bar_y = t.y - (95 * t.scale) - 25

            painter.setBrush(QColor("#e74c3c"))
            painter.setPen(QPen(Qt.GlobalColor.black, 1))
            painter.drawRoundedRect(int(bar_x), int(bar_y), bar_w, bar_h, 3, 3)
            
            hp_width = int((bar_w - 2) * (t.hp / t.max_hp))
            painter.setBrush(QColor("#2ecc71"))
            painter.setPen(Qt.PenStyle.NoPen)
            if hp_width > 0:
                painter.drawRoundedRect(int(bar_x + 1), int(bar_y + 1), hp_width, bar_h - 2, 2, 2)
                
            painter.setPen(Qt.GlobalColor.black)
            painter.setFont(QFont("Arial", 8, QFont.Weight.Bold))
            painter.drawText(int(bar_x), int(bar_y), bar_w, bar_h, Qt.AlignmentFlag.AlignCenter, f"{t.hp}/{t.max_hp}")

    def draw_ui(self, painter):
        painter.setPen(QColor("#2C3E50"))
        painter.setFont(QFont("Arial", 14, QFont.Weight.Bold))
        level_txt = f"УРОВЕНЬ {self.level}/10" if self.state == GameState.LEVEL_MODE else "ТРЕНИРОВКА"
        info = f"{level_txt} | Счёт: {self.score}"
        if self.state == GameState.LEVEL_MODE:
             info += f" | Снежки: {self.snowballs_left}"
             if self.time_left > 0: info += f" | ВРЕМЯ: {self.time_left}"
        painter.drawText(20, 40, info)
        
        painter.setBrush(Qt.BrushStyle.NoBrush)
        painter.setPen(QPen(Qt.GlobalColor.black, 2))
        painter.drawRect(20, 60, 200, 15)
        p_w = int(((self.power-5) / 18) * 200) if self.power > 5 else 0
        painter.fillRect(21, 61, max(0, p_w), 14, QColor("#F1C40F"))

        btn_w = 120
        btn_h = 35
        btn_x = LOGICAL_WIDTH - btn_w - 20
        btn_y = 12

        painter.setBrush(QColor("#e74c3c"))
        painter.setPen(QPen(QColor("#c0392b"), 2))
        painter.drawRoundedRect(btn_x, btn_y, btn_w, btn_h, 6, 6)
        
        painter.setPen(Qt.GlobalColor.white)
        painter.setFont(QFont("Arial", 12, QFont.Weight.Bold))
        painter.drawText(btn_x, btn_y, btn_w, btn_h, Qt.AlignmentFlag.AlignCenter, "ВЫХОД")

    def draw_menu(self, painter):
        # Заголовок игры
        painter.setFont(QFont("Verdana", 40, QFont.Weight.Bold))
        painter.setPen(QColor("#0d1620")) 
        painter.drawText(3, 83, LOGICAL_WIDTH, 100, Qt.AlignmentFlag.AlignCenter, "СНЕЖКИ")
        painter.setPen(Qt.GlobalColor.white) 
        painter.drawText(0, 80, LOGICAL_WIDTH, 100, Qt.AlignmentFlag.AlignCenter, "СНЕЖКИ")
        
        # Общие параметры для кнопок меню
        b_w, b_h = 340, 48
        b_x = (LOGICAL_WIDTH - b_w) // 2
        
        def draw_menu_button(y_pos, text):
            painter.setBrush(QColor("#203245")) 
            painter.setPen(QPen(QColor("#A2D9CE"), 2)) 
            painter.drawRoundedRect(b_x, y_pos, b_w, b_h, 8, 8)
            painter.setPen(QColor("#A2D9CE"))
            painter.setFont(QFont("Verdana", 12, QFont.Weight.Bold))
            painter.drawText(b_x, y_pos, b_w, b_h, Qt.AlignmentFlag.AlignCenter, text)

        # Отрисовка кнопок
        draw_menu_button(210, "ПРАВИЛА ИГРЫ")
        draw_menu_button(275, "НАЧАТЬ ПОХОД (10 УРОВНЕЙ)")
        draw_menu_button(340, "ТРЕНИРОВКА")
        draw_menu_button(405, "ВЫХОД")

        # Окно с правилами игры (Максимально расширенное окно)
        if self.showing_rules:
            painter.fillRect(0, 0, LOGICAL_WIDTH, LOGICAL_HEIGHT, QColor(10, 20, 30, 190))
            
            # Новая ширина 1100 обеспечивает огромный запас под любой текст
            panel_w, panel_h = 1100, 460
            panel_x = (LOGICAL_WIDTH - panel_w) // 2
            panel_y = (LOGICAL_HEIGHT - panel_h) // 2
            
            painter.setBrush(QColor("#152535"))
            painter.setPen(QPen(QColor("#A2D9CE"), 2))
            painter.drawRoundedRect(panel_x, panel_y, panel_w, panel_h, 14, 14)
            
            painter.setPen(Qt.GlobalColor.white)
            painter.setFont(QFont("Verdana", 16, QFont.Weight.Bold))
            painter.drawText(panel_x, panel_y + 35, panel_w, 40, Qt.AlignmentFlag.AlignCenter, "СПРАВКА И УПРАВЛЕНИЕ")
            
            painter.setFont(QFont("Verdana", 12))
            rules_text = (
                "•  МЫШЬ : Свободное прицеливание (изменяет угол направления броска)\n\n"
                "•  ЗАЖАТЬ ПРОБЕЛ : Накопление силы броска (индикатор отображается слева сверху)\n\n"
                "•  ОТПУСТИТЬ ПРОБЕЛ : Совершить бросок снежка по траектории\n\n"
                "•  МИССИЯ : Сбить всех коварных и прочных снеговиков на игровой локации\n\n"
                "•  ПОРАЖЕНИЕ : Закончился лимит доступных снарядов-снежков или полностью истекло время"
            )
            painter.setPen(QColor("#D1F2EB")) 
            painter.drawText(panel_x + 50, panel_y + 110, panel_w - 100, panel_h - 180, Qt.AlignmentFlag.AlignLeft, rules_text)
            
            c_w, c_h = 160, 40
            c_x = (LOGICAL_WIDTH - c_w) // 2
            c_y = panel_y + panel_h - 65
            
            painter.setBrush(QColor("#e74c3c"))
            painter.setPen(QPen(QColor("#c0392b"), 2))
            painter.drawRoundedRect(c_x, c_y, c_w, c_h, 6, 6)
            
            painter.setPen(Qt.GlobalColor.white)
            painter.setFont(QFont("Arial", 11, QFont.Weight.Bold))
            painter.drawText(c_x, c_y, c_w, c_h, Qt.AlignmentFlag.AlignCenter, "ЗАКРЫТЬ")

    def draw_final_screen(self, painter):
        painter.setPen(Qt.GlobalColor.yellow)
        if self.state == GameState.VICTORY:
            painter.setFont(QFont("Arial", 40, QFont.Weight.Bold))
            msg = "ПОБЕДА!\nСНЕЖНЫЙ КОРОЛЬ ПОВЕРЖЕН!"
        else:
            painter.setFont(QFont("Arial", 30, QFont.Weight.Bold))
            msg = "ЗАМЕРЗЛИ РУКИ, ТЫ ПРОИГРАЛ\nПОПРОБУЙ ЗАНОВО"
        
        painter.drawText(0, 0, LOGICAL_WIDTH, LOGICAL_HEIGHT, Qt.AlignmentFlag.AlignCenter, 
                         f"{msg}\n\nСчёт: {self.score}\n\nSPACE - Повторить\nESC - В меню")

    def mouseMoveEvent(self, event):
        scale, off_x, off_y = self.get_scaling()
        lx = (event.position().x() - off_x) / scale
        ly = (event.position().y() - off_y) / scale
        self.angle = math.degrees(math.atan2((GROUND_Y - 45) - ly, lx - (self.player_x + 15)))

    def mousePressEvent(self, event):
        if event.button() == Qt.MouseButton.LeftButton:
            scale, off_x, off_y = self.get_scaling()
            lx = (event.position().x() - off_x) / scale
            ly = (event.position().y() - off_y) / scale

            if self.state == GameState.MENU:
                if self.showing_rules:
                    # Клик по кнопке "ЗАКРЫТЬ" с учётом новой ширины 1100
                    panel_w, panel_h = 1100, 460
                    panel_y = (LOGICAL_HEIGHT - panel_h) // 2
                    c_w, c_h = 160, 40
                    c_x = (LOGICAL_WIDTH - c_w) // 2
                    c_y = panel_y + panel_h - 65
                    if (c_x <= lx <= c_x + c_w) and (c_y <= ly <= c_y + c_h):
                        self.showing_rules = False
                    return 
                else:
                    b_w, b_h = 340, 48
                    b_x = (LOGICAL_WIDTH - b_w) // 2

                    if b_x <= lx <= b_x + b_w:
                        if 210 <= ly <= 210 + b_h:
                            self.showing_rules = True
                        elif 275 <= ly <= 275 + b_h:
                            self.start_level(1)
                        elif 340 <= ly <= 340 + b_h:
                            self.start_training()
                        elif 405 <= ly <= 405 + b_h:
                            sys.exit()

            elif self.state in [GameState.LEVEL_MODE, GameState.TRAINING_MODE]:
                btn_w = 120
                btn_h = 35
                btn_x = LOGICAL_WIDTH - btn_w - 20
                btn_y = 12
                if (btn_x <= lx <= btn_x + btn_w) and (btn_y <= ly <= btn_y + btn_h):
                    self.state = GameState.MENU

            elif self.state in [GameState.GAME_OVER, GameState.VICTORY]:
                if 350 <= lx <= 850:
                    if 380 <= ly <= 480:
                        self.init_variables()
                        self.start_level(1)
                    elif 480 <= ly <= 580:
                        self.state = GameState.MENU

    def keyPressEvent(self, event):
        if event.key() == Qt.Key.Key_Escape:
            if self.state == GameState.MENU:
                if self.showing_rules:
                    self.showing_rules = False  
                else:
                    sys.exit()
            else:
                self.state = GameState.MENU
                self.showing_rules = False
        
        if self.state == GameState.MENU and not self.showing_rules:
            if event.key() == Qt.Key.Key_1: self.start_level(1)
            elif event.key() == Qt.Key.Key_2: self.start_training()
        elif self.state in [GameState.LEVEL_MODE, GameState.TRAINING_MODE]:
            if event.key() == Qt.Key.Key_Space and not event.isAutoRepeat():
                self.is_charging = True
                self.power_phase = 0
        elif self.state in [GameState.GAME_OVER, GameState.VICTORY]:
            if event.key() == Qt.Key.Key_Space: 
                self.init_variables()
                self.start_level(1)

    def keyReleaseEvent(self, event):
        if event.key() == Qt.Key.Key_Space and self.is_charging:
            self.is_charging = False
            if self.snowballs_left > 0:
                if self.state == GameState.LEVEL_MODE: self.snowballs_left -= 1
                self.snowballs.append(Snowball(self.player_x + 15, GROUND_Y - 45, self.angle, self.power))
            self.power = 0

class MainWindow(QMainWindow):
    def __init__(self):
        super().__init__()
        self.setCentralWidget(GameWidget())
        self.setWindowTitle("Snowball Epic Battle")
        self.resize(1100, 600)

if __name__ == "__main__":
    app = QApplication(sys.argv)
    window = MainWindow()
    window.show()
    sys.exit(app.exec())
