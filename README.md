import { Switch, Route } from "wouter";
import { queryClient } from "./lib/queryClient";
import { QueryClientProvider } from "@tanstack/react-query";
import { Toaster } from "@/components/ui/toaster";
import { TooltipProvider } from "@/components/ui/tooltip";
import NotFound from "@/pages/not-found";
import Home from "@/pages/Home";
import GamePlayer from "@/pages/GamePlayer";
function Router() {
  return (
    <Switch>
      <Route path="/" component={Home} />
      <Route path="/game/:id" component={GamePlayer} />
      <Route component={NotFound} />
    </Switch>
  );
}
function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <TooltipProvider>
        <Toaster />
        <Router />
      </TooltipProvider>
    </QueryClientProvider>
  );
}
export default App;
import { sql } from "drizzle-orm";
import { pgTable, text, varchar, integer } from "drizzle-orm/pg-core";
import { createInsertSchema } from "drizzle-zod";
import { z } from "zod";
export const users = pgTable("users", {
  id: varchar("id").primaryKey().default(sql`gen_random_uuid()`),
  username: text("username").notNull().unique(),
  password: text("password").notNull(),
});
export const insertUserSchema = createInsertSchema(users).pick({
  username: true,
  password: true,
});
export type InsertUser = z.infer<typeof insertUserSchema>;
export type User = typeof users.$inferSelect;
export type Game = {
  id: string;
  title: string;
  description: string;
  category: string;
  thumbnail: string;
  embedUrl: string;
  plays: number;
};
export const categories = [
  "All",
  "Action",
  "Puzzle",
  "Arcade",
  "Sports",
  "Racing",
] as const;
export type Category = typeof categories[number];
import { type Game } from "@shared/schema";
import game2048 from "@assets/generated_images/2048_puzzle_game_thumbnail_29f400fa.png";
import snakeGame from "@assets/generated_images/Snake_game_thumbnail_27e4237c.png";
import tetrisGame from "@assets/generated_images/Tetris_blocks_game_thumbnail_7a0e7552.png";
import flappyGame from "@assets/generated_images/Flappy_bird_game_thumbnail_c3acd90f.png";
import racingGame from "@assets/generated_images/Racing_game_thumbnail_2675060b.png";
import spaceGame from "@assets/generated_images/Space_shooter_game_thumbnail_3c301781.png";
import bubbleGame from "@assets/generated_images/Bubble_shooter_game_thumbnail_0dbe05ca.png";
import soccerGame from "@assets/generated_images/Soccer_game_thumbnail_05524ed6.png";
export const games: Game[] = [
  {
    id: "2048",
    title: "2048",
    description: "Combine tiles to reach 2048! A challenging number puzzle game.",
    category: "Puzzle",
    thumbnail: game2048,
    embedUrl: "https://play2048.co/",
    plays: 12500,
  },
  {
    id: "snake",
    title: "Classic Snake",
    description: "Eat apples and grow your snake without hitting the walls!",
    category: "Arcade",
    thumbnail: snakeGame,
    embedUrl: "https://playsnake.org/",
    plays: 8900,
  },
  {
    id: "tetris",
    title: "Tetris Blocks",
    description: "Clear lines by arranging falling blocks in this timeless classic.",
    category: "Puzzle",
    thumbnail: tetrisGame,
    embedUrl: "https://tetris.com/play-tetris",
    plays: 15200,
  },
  {
    id: "flappy",
    title: "Flappy Bird",
    description: "Navigate through pipes in this addictive one-tap arcade game.",
    category: "Arcade",
    thumbnail: flappyGame,
    embedUrl: "https://flappybird.io/",
    plays: 9800,
  },
  {
    id: "racing",
    title: "Neon Racer",
    description: "Race through neon tracks at high speeds. Can you beat your best time?",
    category: "Racing",
    thumbnail: racingGame,
    embedUrl: "https://poki.com/en/g/madalin-stunt-cars-2",
    plays: 11400,
  },
  {
    id: "space",
    title: "Space Invaders",
    description: "Defend Earth from alien invaders in this retro shooter classic.",
    category: "Action",
    thumbnail: spaceGame,
    embedUrl: "https://freeinvaders.org/",
    plays: 7600,
  },
  {
    id: "bubble",
    title: "Bubble Shooter",
    description: "Pop bubbles by matching three or more of the same color!",
    category: "Puzzle",
    thumbnail: bubbleGame,
    embedUrl: "https://www.bubbleshooter.com/",
    plays: 13100,
  },
  {
    id: "soccer",
    title: "Penalty Kicks",
    description: "Score goals and become the penalty shootout champion!",
    category: "Sports",
    thumbnail: soccerGame,
    embedUrl: "https://poki.com/en/g/penalty-shooters-2",
    plays: 6700,
  },
];
import { useState, useMemo } from "react";
import { useLocation } from "wouter";
import Header from "@/components/Header";
import Hero from "@/components/Hero";
import CategoryFilter from "@/components/CategoryFilter";
import GameGrid from "@/components/GameGrid";
import { games } from "@/lib/games";
import { type Category } from "@shared/schema";
export default function Home() {
  const [, setLocation] = useLocation();
  const [searchValue, setSearchValue] = useState("");
  const [selectedCategory, setSelectedCategory] = useState<Category>("All");
  const filteredGames = useMemo(() => {
    return games.filter((game) => {
      const matchesSearch = game.title
        .toLowerCase()
        .includes(searchValue.toLowerCase());
      const matchesCategory =
        selectedCategory === "All" || game.category === selectedCategory;
      return matchesSearch && matchesCategory;
    });
  }, [searchValue, selectedCategory]);
  const scrollToGames = () => {
    const gamesSection = document.getElementById("games-section");
    gamesSection?.scrollIntoView({ behavior: "smooth" });
  };
  return (
    <div className="min-h-screen bg-background">
      <Header searchValue={searchValue} onSearchChange={setSearchValue} />
      <main className="container mx-auto px-4 py-8">
        <Hero onPlayClick={scrollToGames} />
        <div id="games-section" className="mt-12">
          <div className="mb-6">
            <h2 className="text-3xl font-bold mb-4" data-testid="text-browse-title">
              Browse Games
            </h2>
            <CategoryFilter
              selectedCategory={selectedCategory}
              onCategoryChange={setSelectedCategory}
            />
          </div>
          <GameGrid
            games={filteredGames}
            onGameClick={(game) => setLocation(`/game/${game.id}`)}
          />
        </div>
      </main>
      <footer className="border-t mt-16 py-8">
        <div className="container mx-auto px-4 text-center text-muted-foreground">
          <p data-testid="text-footer">© 2024 GameHub. All games are property of their respective owners.</p>
        </div>
      </footer>
    </div>
  );
}
import { useLocation, useRoute, Link } from "wouter";
import { ArrowLeft, Maximize2 } from "lucide-react";
import { Button } from "@/components/ui/button";
import { Card } from "@/components/ui/card";
import Header from "@/components/Header";
import GameGrid from "@/components/GameGrid";
import { games } from "@/lib/games";
import { useState } from "react";
export default function GamePlayer() {
  const [, setLocation] = useLocation();
  const [, params] = useRoute("/game/:id");
  const [searchValue, setSearchValue] = useState("");
  const game = games.find((g) => g.id === params?.id);
  if (!game) {
    return (
      <div className="min-h-screen bg-background">
        <Header searchValue={searchValue} onSearchChange={setSearchValue} />
        <div className="container mx-auto px-4 py-16 text-center">
          <h1 className="text-2xl font-bold mb-4">Game not found</h1>
          <Link href="/">
            <Button>Back to Home</Button>
          </Link>
        </div>
      </div>
    );
  }
  const relatedGames = games
    .filter((g) => g.category === game.category && g.id !== game.id)
    .slice(0, 4);
  return (
    <div className="min-h-screen bg-background">
      <Header searchValue={searchValue} onSearchChange={setSearchValue} />
      <main className="container mx-auto px-4 py-8">
        <div className="mb-6">
          <Button
            variant="ghost"
            className="gap-2"
            onClick={() => setLocation("/")}
            data-testid="button-back"
          >
            <ArrowLeft className="w-4 h-4" />
            Back to Games
          </Button>
        </div>
        <Card className="p-0 overflow-hidden mb-8">
          <div className="aspect-video w-full bg-muted relative">
            <iframe
              src={game.embedUrl}
              className="w-full h-full"
              title={game.title}
              allowFullScreen
              data-testid="iframe-game"
            />
          </div>
        </Card>
        <div className="mb-12">
          <div className="flex items-start justify-between gap-4 mb-4">
            <div>
              <h1 className="text-3xl font-bold mb-2" data-testid="text-game-title">
                {game.title}
              </h1>
              <p className="text-muted-foreground" data-testid="text-game-category">
                {game.category}
              </p>
            </div>
            <Button variant="outline" size="icon" data-testid="button-fullscreen">
              <Maximize2 className="w-4 h-4" />
            </Button>
          </div>
          <p className="text-foreground" data-testid="text-game-description">
            {game.description}
          </p>
        </div>
        {relatedGames.length > 0 && (
          <div>
            <h2 className="text-2xl font-bold mb-6" data-testid="text-related-title">
              More {game.category} Games
            </h2>
            <GameGrid
              games={relatedGames}
              onGameClick={(g) => setLocation(`/game/${g.id}`)}
            />
          </div>
        )}
      </main>
    </div>
  );
}

5. Components
Header.tsx
import { Gamepad2, Moon, Sun } from "lucide-react";
import { Button } from "@/components/ui/button";
import SearchBar from "./SearchBar";
import { Link } from "wouter";
import { useEffect, useState } from "react";
interface HeaderProps {
  searchValue: string;
  onSearchChange: (value: string) => void;
}
export default function Header({ searchValue, onSearchChange }: HeaderProps) {
  const [theme, setTheme] = useState<"light" | "dark">("light");
  useEffect(() => {
    const savedTheme = localStorage.getItem("theme") as "light" | "dark" | null;
    const initialTheme = savedTheme || "light";
    setTheme(initialTheme);
    document.documentElement.classList.toggle("dark", initialTheme === "dark");
  }, []);
  const toggleTheme = () => {
    const newTheme = theme === "light" ? "dark" : "light";
    setTheme(newTheme);
    localStorage.setItem("theme", newTheme);
    document.documentElement.classList.toggle("dark", newTheme === "dark");
  };
  return (
    <header className="sticky top-0 z-50 w-full border-b bg-background/95 backdrop-blur supports-[backdrop-filter]:bg-background/60">
      <div className="container mx-auto px-4 h-16 flex items-center justify-between gap-4">
        <Link href="/" data-testid="link-home">
          <div className="flex items-center gap-2 cursor-pointer">
            <Gamepad2 className="w-6 h-6 text-primary" data-testid="icon-logo" />
            <span className="font-bold text-xl">GameHub</span>
          </div>
        </Link>
        <div className="flex-1 max-w-md hidden md:flex">
          <SearchBar value={searchValue} onChange={onSearchChange} />
        </div>
        <Button
          variant="ghost"
          size="icon"
          onClick={toggleTheme}
          data-testid="button-theme-toggle"
        >
          {theme === "light" ? (
            <Moon className="w-5 h-5" />
          ) : (
            <Sun className="w-5 h-5" />
          )}
        </Button>
      </div>
      <div className="container mx-auto px-4 pb-3 md:hidden">
        <SearchBar value={searchValue} onChange={onSearchChange} />
      </div>
    </header>
  );
}
import { Button } from "@/components/ui/button";
import { Play } from "lucide-react";
import heroBanner from "@assets/generated_images/Gaming_hero_banner_montage_97d43b56.png";
interface HeroProps {
  onPlayClick: () => void;
}
export default function Hero({ onPlayClick }: HeroProps) {
  return (
    <div className="relative w-full h-[400px] md:h-[500px] overflow-hidden rounded-lg">
      <div
        className="absolute inset-0 bg-cover bg-center"
        style={{ backgroundImage: `url(${heroBanner})` }}
      />
      <div className="absolute inset-0 bg-gradient-to-r from-black/70 via-black/50 to-black/30" />
      
      <div className="relative h-full flex flex-col items-center justify-center text-center px-4">
        <h1 className="text-4xl md:text-6xl font-bold text-white mb-4" data-testid="text-hero-title">
          Play Free Games Online
        </h1>
        <p className="text-lg md:text-xl text-white/90 mb-8 max-w-2xl" data-testid="text-hero-subtitle">
          No downloads, no ads, just pure fun. Choose from hundreds of unblocked games!
        </p>
        <Button
          size="lg"
          variant="default"
          className="gap-2"
          onClick={onPlayClick}
          data-testid="button-hero-play"
        >
          <Play className="w-5 h-5" />
          Start Playing
        </Button>
      </div>
    </div>
  );
}

GameCard.tsx
import { Card } from "@/components/ui/card";
import { Badge } from "@/components/ui/badge";
import { Eye } from "lucide-react";
import { type Game } from "@shared/schema";
interface GameCardProps {
  game: Game;
  onClick: () => void;
}
export default function GameCard({ game, onClick }: GameCardProps) {
  return (
    <Card
      className="overflow-hidden cursor-pointer transition-transform hover:scale-105 hover-elevate active-elevate-2"
      onClick={onClick}
      data-testid={`card-game-${game.id}`}
    >
      <div className="aspect-square relative overflow-hidden">
        <img
          src={game.thumbnail}
          alt={game.title}
          className="w-full h-full object-cover"
          data-testid={`img-thumbnail-${game.id}`}
        />
        <div className="absolute top-2 right-2">
          <Badge variant="secondary" className="gap-1" data-testid={`badge-plays-${game.id}`}>
            <Eye className="w-3 h-3" />
            {game.plays.toLocaleString()}
          </Badge>
        </div>
      </div>
      <div className="p-4">
        <h3 className="font-semibold text-lg mb-1" data-testid={`text-title-${game.id}`}>
          {game.title}
        </h3>
        <p className="text-sm text-muted-foreground line-clamp-2" data-testid={`text-description-${game.id}`}>
          {game.description}
        </p>
        <div className="mt-2">
          <Badge variant="outline" data-testid={`badge-category-${game.id}`}>
            {game.category}
          </Badge>
        </div>
      </div>
    </Card>
  );
}
import { type Game } from "@shared/schema";
import GameCard from "./GameCard";
interface GameGridProps {
  games: Game[];
  onGameClick: (game: Game) => void;
}
export default function GameGrid({ games, onGameClick }: GameGridProps) {
  if (games.length === 0) {
    return (
      <div className="text-center py-16">
        <p className="text-muted-foreground text-lg" data-testid="text-no-games">
          No games found. Try a different search or category.
        </p>
      </div>
    );
  }
  return (
    <div className="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 xl:grid-cols-5 gap-4 md:gap-6" data-testid="grid-games">
      {games.map((game) => (
        <GameCard key={game.id} game={game} onClick={() => onGameClick(game)} />
      ))}
    </div>
  );
}
import { Button } from "@/components/ui/button";
import { categories, type Category } from "@shared/schema";
interface CategoryFilterProps {
  selectedCategory: Category;
  onCategoryChange: (category: Category) => void;
}
export default function CategoryFilter({
  selectedCategory,
  onCategoryChange,
}: CategoryFilterProps) {
  return (
    <div className="flex gap-2 overflow-x-auto pb-2 scrollbar-hide" data-testid="filter-categories">
      {categories.map((category) => (
        <Button
          key={category}
          variant={selectedCategory === category ? "default" : "outline"}
          className="whitespace-nowrap toggle-elevate"
          onClick={() => onCategoryChange(category)}
          data-testid={`button-category-${category.toLowerCase()}`}
        >
          {category}
        </Button>
      ))}
    </div>
  );
}
import { Input } from "@/components/ui/input";
import { Search } from "lucide-react";
interface SearchBarProps {
  value: string;
  onChange: (value: string) => void;
  placeholder?: string;
}
export default function SearchBar({
  value,
  onChange,
  placeholder = "Search games...",
}: SearchBarProps) {
  return (
    <div className="relative w-full max-w-md">
      <Search className="absolute left-3 top-1/2 transform -translate-y-1/2 w-4 h-4 text-muted-foreground" />
      <Input
        type="search"
        placeholder={placeholder}
        value={value}
        onChange={(e) => onChange(e.target.value)}
        className="pl-10"
        data-testid="input-search"
      />
    </div>
  );
}
