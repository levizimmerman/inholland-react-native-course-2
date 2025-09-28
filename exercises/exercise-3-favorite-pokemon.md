# Exercise 3: Favorite Pokemon with Local Database

In this exercise, you'll learn how to implement a favorites system for your Pokédex app using Expo SQLite for local data persistence. You'll create a database service, favorite toggle component, and a favorites page.

## Prerequisites

- Completed Exercise 1 (TanStack Query setup)
- Completed Exercise 2 (Pokemon detail pages)
- Basic understanding of SQLite databases
- Familiarity with React Native components

## Step 1: Install Required Dependencies

First, let's install the necessary packages for local database functionality.

```bash
# Install Expo SQLite for local database
npx expo install expo-sqlite
```

## Step 2: Create Database Service

Create a new file `services/database.ts` to handle all database operations:

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

  async removeFavorite(pokemonId: number): Promise<void> {
    if (!this.db) throw new Error("Database not initialized");

    try {
      await this.db.runAsync("DELETE FROM favorites WHERE id = ?", [pokemonId]);
    } catch (error) {
      console.error("Error removing favorite:", error);
      throw error;
    }
  }

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
}

export const databaseService = new DatabaseService();
```

## Step 3: Initialize Database in App Layout

Update your root layout file (`app/_layout.tsx`) to initialize the database when the app starts:

```typescript
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { Stack } from 'expo-router';
import { SafeAreaProvider } from 'react-native-safe-area-context';
import { useEffect } from 'react';
import { databaseService } from '@/services/database';

// Create a QueryClient instance with default options
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000, // 5 minutes
      gcTime: 10 * 60 * 1000, // 10 minutes
    },
  },
});

export default function RootLayout() {
  useEffect(() => {
    // Initialize database when app starts
    databaseService.initDatabase().catch(console.error);
  }, []);

  return (
    <QueryClientProvider client={queryClient}>
      <SafeAreaProvider>
        <Stack>
          <Stack.Screen name="(tabs)" options={{ headerShown: false }} />
          <Stack.Screen name="pokemon/[name]" options={{ title: "Pokemon Details" }} />
        </Stack>
      </SafeAreaProvider>
    </QueryClientProvider>
  );
}
```

## Step 4: Create Favorites Hooks

Create a new file `hooks/use-favorites.ts` to manage favorite Pokemon data:

```typescript
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { databaseService } from '@/services/database';

// Hook to get all favorite Pokemon
export const useFavorites = () => {
  return useQuery({
    queryKey: ['favorites'],
    queryFn: () => databaseService.getAllFavorites(),
    staleTime: 0, // Always fetch fresh data for favorites
  });
};

// Hook to check if a Pokemon is favorited
export const useIsFavorite = (pokemonId: number) => {
  return useQuery({
    queryKey: ['is-favorite', pokemonId],
    queryFn: () => databaseService.isFavorite(pokemonId),
    staleTime: 0,
  });
};

// Hook to toggle favorite status
export const useToggleFavorite = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: async ({ 
      pokemonId, 
      name, 
      imageUrl, 
      isCurrentlyFavorite 
    }: {
      pokemonId: number;
      name: string;
      imageUrl?: string;
      isCurrentlyFavorite: boolean;
    }) => {
      if (isCurrentlyFavorite) {
        await databaseService.removeFavorite(pokemonId);
      } else {
        await databaseService.addFavorite(pokemonId, name, imageUrl);
      }
    },
    onSuccess: (_, variables) => {
      // Invalidate and refetch favorites
      queryClient.invalidateQueries({ queryKey: ['favorites'] });
      queryClient.invalidateQueries({ queryKey: ['is-favorite', variables.pokemonId] });
    },
  });
};
```

## Step 5: Create Favorite Toggle Component

Create a new file `components/ui/favorite.tsx` for the favorite toggle button:

```typescript
import { useIsFavorite } from '@/hooks/use-favorites';
import { useToggleFavorite } from '@/hooks/use-favorites';
import { Ionicons } from '@expo/vector-icons';
import { TouchableOpacity, StyleSheet } from 'react-native';

interface FavoriteProps {
  pokemonId: number;
  pokemonName: string;
  imageUrl?: string;
}

export default function Favorite({ pokemonId, pokemonName, imageUrl }: FavoriteProps) {
  const { data: isFavorited, isLoading } = useIsFavorite(pokemonId);
  const toggleFavorite = useToggleFavorite();

  const handleToggle = () => {
    if (isLoading) return;

    toggleFavorite.mutate({
      pokemonId,
      name: pokemonName,
      imageUrl,
      isCurrentlyFavorite: isFavorited || false,
    });
  };

  return (
    <TouchableOpacity
      style={styles.favoriteButton}
      onPress={handleToggle}
      disabled={toggleFavorite.isPending}
    >
      <Ionicons
        name={isFavorited ? "heart" : "heart-outline"}
        size={24}
        color={isFavorited ? "#FF6B6B" : "#666"}
      />
    </TouchableOpacity>
  );
}

const styles = StyleSheet.create({
  favoriteButton: {
    padding: 8,
    borderRadius: 20,
    backgroundColor: 'rgba(255, 255, 255, 0.9)',
  },
});
```

## Step 6: Add Favorite Button to Pokemon Cards

Update your Pokemon card component to include the favorite button. In your `components/ui/pokemon-card.tsx`:

```diff
export default function PokemonCard({ pokemon, onPress }: PokemonCardProps) {
  const imageUrl = `https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/official-artwork/${pokemon.id}.png`;

  return (
    <TouchableOpacity style={styles.card} onPress={onPress}>
      <View style={styles.imageContainer}>
        <PokemonImage id={pokemon.id} size={120} />
      </View>
      
      <View style={styles.infoContainer}>
        <Text style={styles.pokemonName}>
          {pokemon.name.charAt(0).toUpperCase() + pokemon.name.slice(1)}
        </Text>
        <Text style={styles.pokemonId}>#{pokemon.id.padStart(3, '0')}</Text>
      </View>
      
+     <View style={styles.favoriteContainer}>
+       <Favorite
+         pokemonId={parseInt(pokemon.id)}
+         pokemonName={pokemon.name}
+         imageUrl={imageUrl}
+       />
+     </View>
    </TouchableOpacity>
  );
}
```

## Step 7: Create Favorites Page

Create a new file `app/(tabs)/favorites.tsx` for the favorites tab:

```typescript
import { useFavorites } from '@/hooks/use-favorites';
import { ActivityIndicator, FlatList, StyleSheet, Text, View } from 'react-native';
import { SafeAreaView } from 'react-native-safe-area-context';
import PokemonCard from '@/components/ui/pokemon-card';
import { router } from 'expo-router';

export default function FavoritesScreen() {
  const { data: favorites, isLoading, error } = useFavorites();

  const handlePokemonPress = (pokemonName: string) => {
    router.push(`/pokemon/${pokemonName}`);
  };

  const renderPokemonCard = ({ item }: { item: any }) => (
    <PokemonCard
      pokemon={{
        id: item.id.toString(),
        name: item.name,
        url: '', // Not needed for favorites
      }}
      onPress={() => handlePokemonPress(item.name)}
    />
  );

  if (isLoading) {
    return (
      <SafeAreaView style={styles.container}>
        <View style={styles.header}>
          <Text style={styles.title}>My Favorites</Text>
        </View>
        <View style={styles.loadingContainer}>
          <ActivityIndicator size="large" color="#5631E8" />
          <Text style={styles.loadingText}>Loading favorites...</Text>
        </View>
      </SafeAreaView>
    );
  }

  if (error || !favorites || favorites.length === 0) {
    return (
      <SafeAreaView style={styles.container}>
        <View style={styles.header}>
          <Text style={styles.title}>My Favorites</Text>
        </View>
        <View style={styles.emptyContainer}>
          <Text style={styles.emptyText}>No favorites yet</Text>
          <Text style={styles.emptySubtext}>
            Tap the heart icon on any Pokémon to add it to your favorites!
          </Text>
        </View>
      </SafeAreaView>
    );
  }

  return (
    <SafeAreaView style={styles.container}>
      <View style={styles.header}>
        <Text style={styles.title}>My Favorites</Text>
        <Text style={styles.subtitle}>
          {favorites.length} {favorites.length === 1 ? 'Pokémon' : 'Pokémon'} saved
        </Text>
      </View>
      
      <FlatList
        data={favorites}
        renderItem={renderPokemonCard}
        keyExtractor={(item) => item.id.toString()}
        contentContainerStyle={styles.listContainer}
        showsVerticalScrollIndicator={false}
      />
    </SafeAreaView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f0f8ff',
  },
  header: {
    paddingHorizontal: 16,
    paddingVertical: 16,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#0E0940',
    marginBottom: 4,
  },
  subtitle: {
    fontSize: 16,
    color: '#666',
  },
  loadingContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 20,
  },
  loadingText: {
    marginTop: 10,
    fontSize: 16,
    color: '#5631E8',
  },
  emptyContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 40,
  },
  emptyText: {
    fontSize: 20,
    fontWeight: 'bold',
    color: '#666',
    marginBottom: 8,
    textAlign: 'center',
  },
  emptySubtext: {
    fontSize: 16,
    color: '#999',
    textAlign: 'center',
    lineHeight: 22,
  },
  listContainer: {
    paddingHorizontal: 16,
    paddingBottom: 20,
  },
});
```

## Step 8: Update Tab Layout

Make sure your tab layout includes the favorites tab. Update `app/(tabs)/_layout.tsx`:

```typescript
import { Tabs } from 'expo-router';
import { Ionicons } from '@expo/vector-icons';

export default function TabLayout() {
  return (
    <Tabs
      screenOptions={{
        tabBarActiveTintColor: '#5631E8',
        tabBarInactiveTintColor: '#666',
        headerShown: false,
      }}
    >
      <Tabs.Screen
        name="index"
        options={{
          title: 'Pokemon',
          tabBarIcon: ({ color, size }) => (
            <Ionicons name="list" size={size} color={color} />
          ),
        }}
      />
      <Tabs.Screen
        name="favorites"
        options={{
          title: 'Favorites',
          tabBarIcon: ({ color, size }) => (
            <Ionicons name="heart" size={size} color={color} />
          ),
        }}
      />
    </Tabs>
  );
}
```

## Step 9: Test Your Implementation

1. Start your development server:
   ```bash
   npm run start
   ```

2. Test the following functionality:
   - Navigate to the Pokemon list
   - Tap the heart icon on any Pokemon to add/remove from favorites
   - Navigate to the Favorites tab to see your saved Pokemon
   - Tap on a favorite Pokemon to view its details
   - Verify that the heart icon shows the correct state (filled/outline)

## Key Concepts Learned

- **SQLite Database**: Local data persistence with Expo SQLite
- **Database Service**: Centralized database operations
- **React Query Mutations**: Managing database write operations
- **Query Invalidation**: Keeping UI in sync with database changes
- **Local State Management**: Combining server state with local state
- **Database Schema**: Creating and managing database tables

## Bonus Features

- Add haptic feedback when toggling favorites
- Add favorite Pokemon statistics

## Troubleshooting

- **Database not initialized**: Make sure to call `initDatabase()` in your app layout
- **SQLite errors**: Check that all database operations are properly awaited
- **Query not updating**: Ensure you're invalidating the correct query keys
- **Import errors**: Verify all file paths and imports are correct

## Resources

- [Expo SQLite Documentation](https://docs.expo.dev/versions/latest/sdk/sqlite/)
- [TanStack Query Mutations](https://tanstack.com/query/latest/docs/framework/react/reference/useMutation)
- [Expo Haptics](https://docs.expo.dev/versions/latest/sdk/haptics/)
