---
marp: true
theme: default
class: invert
---

# Tanstack Query
## Getting the Pokédex Data

---

# Tanstack Query
## What does it solve?

- Data fetching
- Caching
- Synchronization
- ... and more

![bg right fit](../assets/tanstack.png)

---

![bg fit](../assets/tanstack-arch.png)

---

# Traditional React Fetching

```jsx
function PokemonList() {
  const [pokemon, setPokemon] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchPokemon = async () => {
      try {
        setLoading(true);
        const response = await fetch('https://pokeapi.co/api/v2/pokemon?limit=20');
        const data = await response.json();
        setPokemon(data.results);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };

    fetchPokemon();
  }, []);
}
```

---

# With TanStack Query

```jsx
import { useQuery } from '@tanstack/react-query';

function PokemonList() {
  const { data: pokemon, isLoading, error } = useQuery({
    queryKey: ['pokemon'], // cache identifier
    queryFn: () => 
      fetch('https://pokeapi.co/api/v2/pokemon?limit=20')
        .then(res => res.json())
        .then(data => data.results)
  });
```

---

![bg fit](../assets/tanstack-cache.png)

---

# Query anything

```jsx
useQuery({
  queryKey: ['pokemon', pokemonId],
  // fetching something from the server
  queryFn: () => someApiService.getPokemonDetails(pokemonId),
});
useQuery({
  queryKey: ['favorites'],
  // fetching something from local storage
  queryFn: () => someLocalStorage.getFavorites(),
});
```

---

![bg fit](../assets/tanstack-arch-any.png)

---

# SQLite CRUD Examples
## Create, Read, Update, Delete Operations

---

# Creating Entries (INSERT)

```typescript
// Add a new Pokemon to favorites
async addFavorite(
  pokemonId: number,
  name: string,
  imageUrl?: string
): Promise<void> {
  if (!this.db) throw new Error("Database not initialized");

  try {
    await this.db.runAsync(
      "INSERT OR REPLACE INTO favorites (id, name, image_url) VALUES (?, ?, ?)",
      [pokemonId, name, imageUrl || ""]
    );
  } catch (error) {
    console.error("Error adding favorite:", error);
    throw error;
  }
}

// Usage
await databaseService.addFavorite(25, "pikachu", "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/25.png");
```

---

# Reading Entries (SELECT)

```typescript
// Get all favorites
async getAllFavorites(): Promise<FavoritePokemon[]> {
  if (!this.db) throw new Error("Database not initialized");

  try {
    const result = await this.db.getAllAsync<FavoritePokemon>(
      "SELECT * FROM favorites ORDER BY created_at DESC"
    );
    return result;
  } catch (error) {
    console.error("Error getting favorites:", error);
    return [];
  }
}

// Check if Pokemon is favorite
async isFavorite(pokemonId: number): Promise<boolean> {
  if (!this.db) throw new Error("Database not initialized");

  try {
    const result = await this.db.getFirstAsync<{ count: number }>(
      "SELECT COUNT(*) as count FROM favorites WHERE id = ?",
      [pokemonId]
    );
    return (result?.count || 0) > 0;
  } catch (error) {
    console.error("Error checking favorite status:", error);
    return false;
  }
}

// Usage
const favorites = await databaseService.getAllFavorites();
const isPikachuFavorite = await databaseService.isFavorite(25);
```

---

# Removing Entries (DELETE)

```typescript
// Remove Pokemon from favorites
async removeFavorite(pokemonId: number): Promise<void> {
  if (!this.db) throw new Error("Database not initialized");

  try {
    await this.db.runAsync("DELETE FROM favorites WHERE id = ?", [pokemonId]);
  } catch (error) {
    console.error("Error removing favorite:", error);
    throw error;
  }
}

// Usage
await databaseService.removeFavorite(25);
```

---

# Complete Database Service Example

```typescript
import * as SQLite from "expo-sqlite";

export interface FavoritePokemon {
  id: number;
  name: string;
  image_url: string;
  created_at: string;
}

class DatabaseService {
  private db: SQLite.SQLiteDatabase | null = null;

  async initDatabase(): Promise<void> {
    try {
      this.db = await SQLite.openDatabaseAsync("pokedex.db");
      await this.createTables();
    } catch (error) {
      console.error("Error initializing database:", error);
      throw error;
    }
  }

  private async createTables(): Promise<void> {
    if (!this.db) throw new Error("Database not initialized");

    await this.db.execAsync(`
      CREATE TABLE IF NOT EXISTS favorites (
        id INTEGER PRIMARY KEY,
        name TEXT NOT NULL,
        image_url TEXT,
        created_at DATETIME DEFAULT CURRENT_TIMESTAMP
      );
    `);
  }

  // CRUD methods here...
}

export const databaseService = new DatabaseService();
```
