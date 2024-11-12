```py
# OOP-Project

# 20. Розробка програми обліку спортивних змагань.

from typing import Protocol, List
import pgzrun
import pygame
import random
import json


WIDTH = 800
HEIGHT = 600
background_color = (0, 0, 30)

class Sport(Protocol):
    def get_sport_name(self) -> str:
        pass

    def get_competitions(self) -> List[str]:
        pass


class BaseSport:
    def __init__(self, sport_name: str):
        self.sport_name = sport_name
        self.competitions = self.load_competitions()

    def load_competitions(self) -> List[str]:
        with open("competitions.json", "r", encoding="utf-8") as f:
            data = json.load(f)
            return data.get(self.sport_name, [])

    def get_sport_name(self) -> str:
        return self.sport_name

    def get_competitions(self) -> List[str]:
        return self.competitions


class FootballSport(BaseSport):
    def get_competitions(self) -> List[str]:
        competitions = super().get_competitions()
        return competitions


class BasketballSport(BaseSport):
    def get_competitions(self) -> List[str]:
        competitions = super().get_competitions()
        return competitions


class VolleyballSport(BaseSport):
    def get_competitions(self) -> List[str]:
        competitions = super().get_competitions()
        return competitions


class HockeySport(BaseSport):
    def get_competitions(self) -> List[str]:
        competitions = super().get_competitions()
        return competitions


class TennisSport(BaseSport):
    def get_competitions(self) -> List[str]:
        competitions = super().get_competitions()
        return competitions


class FallingObject:
    def __init__(self, image, x, y, speed, scale_factor=0.3):
        self.image = pygame.image.load(f"images/{image}.png")
        original_size = self.image.get_size()
        new_size = (int(original_size[0] * scale_factor), int(original_size[1] * scale_factor))
        self.image = pygame.transform.scale(self.image, new_size)
        
        self.x = x
        self.y = y
        self.speed = speed

    def fall(self):
        self.y += self.speed
        if self.y > HEIGHT:
            self.y = -50
            self.x = random.randint(0, WIDTH - 50)
            self.speed = random.uniform(1, 3)

    def draw(self):
        screen.blit(self.image, (self.x, self.y))


class SportApp:
    def __init__(self):
        self.background_color = background_color
        self.sports_objects = [
            FootballSport("Футбол"),
            BasketballSport("Баскетбол"),
            VolleyballSport("Волейбол"),
            HockeySport("Хокей"),
            TennisSport("Теніс")
        ]
        self.sports = [sport.get_sport_name() for sport in self.sports_objects]
        self.selected_sport: Sport = None
        self.hovered_sport = None
        self.show_sport_info = False

        self.buttons = []
        for i, sport in enumerate(self.sports):
            self.buttons.append(
                {"sport": sport, "x": WIDTH // 2 - 200, "y": 100 + i * 70, "width": 400, "height": 50}
            )

        self.hover_colors = {
            "Футбол": "#B2E0B2",
            "Баскетбол": "#FFDAB9",
            "Волейбол": "#FFFACD",
            "Хокей": "lightblue",
            "Теніс": "#FFFFE0"
        }

    
        self.falling_objects = [
            FallingObject("football", random.randint(0, WIDTH - 50), random.randint(-600, 0), random.uniform(1, 3), scale_factor=0.3),
            FallingObject("basketball", random.randint(0, WIDTH - 50), random.randint(-600, 0), random.uniform(1, 3), scale_factor=0.3),
            FallingObject("volleyball", random.randint(0, WIDTH - 50), random.randint(-600, 0), random.uniform(1, 3), scale_factor=0.3),
            FallingObject("hockey_puck", random.randint(0, WIDTH - 50), random.randint(-600, 0), random.uniform(1, 3), scale_factor=0.3),
            FallingObject("tennis_ball", random.randint(0, WIDTH - 50), random.randint(-600, 0), random.uniform(1, 3), scale_factor=0.3)
        ]

    def draw(self):
        screen.fill(self.background_color)

        for obj in self.falling_objects:
            obj.draw()

        if not self.show_sport_info:
            self.draw_title()
            self.draw_buttons()
        else:
            self.draw_sport_info()

    def draw_title(self):
        title_text = "Облік спортивних змагань"
        title_surface = pygame.font.Font(None, 60).render(title_text, True, "white")
        title_rect = title_surface.get_rect(center=(WIDTH // 2, 40))
        screen.blit(title_surface, title_rect)

    def draw_buttons(self):
        for button in self.buttons:
            rect_color = "white"

            if self.selected_sport and self.selected_sport.get_sport_name() == button['sport']:
                rect_color = "yellow"
            elif self.hovered_sport == button['sport']:
                rect_color = self.hover_colors.get(button['sport'], "white")

            self.draw_rounded_rect(button["x"], button["y"], button["width"], button["height"], rect_color)
            screen.draw.text(button["sport"], center=(button["x"] + button["width"] // 2, button["y"] + button["height"] // 2), fontsize=40, color="black")

    def draw_sport_info(self):
        if self.selected_sport:
            info_text = f"{self.selected_sport.get_sport_name()}"
            info_surface = pygame.font.Font(None, 50).render(info_text, True, "white")
            info_rect = info_surface.get_rect(center=(WIDTH // 2, 40))
            screen.blit(info_surface, info_rect)

            competitions = self.selected_sport.get_competitions()
            for i, competition in enumerate(competitions):
                screen.draw.text(competition, topleft=(50, 100 + i * 40), fontsize=30, color="white")

    def draw_rounded_rect(self, x, y, width, height, color, radius=15):
        surface = pygame.Surface((width, height), pygame.SRCALPHA)
        pygame.draw.rect(surface, color, (0, 0, width, height), border_radius=radius)
        screen.blit(pygame.transform.scale(surface, (width, height)), (x, y))

    def on_mouse_down(self, pos, button):
        if button == 1:
            if not self.show_sport_info:
                for sport_button in self.buttons:
                    if sport_button["x"] < pos[0] < sport_button["x"] + sport_button["width"] and sport_button["y"] < pos[1] < sport_button["y"] + sport_button["height"]:
                        self.selected_sport = next(sport for sport in self.sports_objects if sport.get_sport_name() == sport_button['sport'])
                        self.show_sport_info = True
                        self.update_falling_objects(self.selected_sport.get_sport_name())
                        break
        elif button == 3:
            self.selected_sport = None
            self.show_sport_info = False
            self.update_falling_objects("all")

    def update_falling_objects(self, sport_name):
        self.falling_objects = []

        for _ in range(3):
            if sport_name == "Футбол":
                self.falling_objects.append(FallingObject("football", random.randint(0, WIDTH - 50), random.randint(-600, 0), random.uniform(1, 3), scale_factor=0.3))
            elif sport_name == "Баскетбол":
                self.falling_objects.append(FallingObject("basketball", random.randint(0, WIDTH - 50), random.randint(-600, 0), random.uniform(1, 3), scale_factor=0.3))
            elif sport_name == "Волейбол":
                self.falling_objects.append(FallingObject("volleyball", random.randint(0, WIDTH - 50), random.randint(-600, 0), random.uniform(1, 3), scale_factor=0.3))
            elif sport_name == "Хокей":
                self.falling_objects.append(FallingObject("hockey_puck", random.randint(0, WIDTH - 50), random.randint(-600, 0), random.uniform(1, 3), scale_factor=0.3))
            elif sport_name == "Теніс":
                self.falling_objects.append(FallingObject("tennis_ball", random.randint(0, WIDTH - 50), random.randint(-600, 0), random.uniform(1, 3), scale_factor=0.3))
            elif sport_name == "all":
                self.falling_objects.extend([
                    FallingObject("football", random.randint(0, WIDTH - 50), random.randint(-600, 0), random.uniform(1, 3), scale_factor=0.3),
                    FallingObject("basketball", random.randint(0, WIDTH - 50), random.randint(-600, 0), random.uniform(1, 3), scale_factor=0.3),
                    FallingObject("volleyball", random.randint(0, WIDTH - 50), random.randint(-600, 0), random.uniform(1, 3), scale_factor=0.3),
                    FallingObject("hockey_puck", random.randint(0, WIDTH - 50), random.randint(-600, 0), random.uniform(1, 3), scale_factor=0.3),
                    FallingObject("tennis_ball", random.randint(0, WIDTH - 50), random.randint(-600, 0), random.uniform(1, 3), scale_factor=0.3)
                ])

    def on_mouse_move(self, pos):
        if not self.show_sport_info:
            self.hovered_sport = None
            for button in self.buttons:
                if button["x"] < pos[0] < button["x"] + button["width"] and button["y"] < pos[1] < button["y"] + button["height"]:
                    self.hovered_sport = button['sport']
                    break

    def update(self):
        for obj in self.falling_objects:
            obj.fall()

app = SportApp()

def update():
    app.update()

def draw():
    app.draw()

def on_mouse_down(pos, button):
    app.on_mouse_down(pos, button)

def on_mouse_move(pos):
    app.on_mouse_move(pos)

pgzrun.go()
```
