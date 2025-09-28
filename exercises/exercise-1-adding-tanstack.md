# Exercise 1: Adding TanStack Query to Your Pokédex App

In this exercise, you'll learn how to integrate TanStack Query into your React Native Pokédex app to efficiently manage server state and data fetching.

## Prerequisites

- A React Native Expo project set up
- Basic understanding of React hooks
- Familiarity with TypeScript

## Step 1: Install Required Dependencies

First, let's install the necessary packages for data fetching and state management.

```bash
# Install TanStack Query
npm install @tanstack/react-query

# Install pokenode-ts for easy Pokemon API integration
npm install pokenode-ts
```

## Step 2: Set Up QueryClient Provider

Create or update your root layout file (`app/_layout.tsx`) to wrap your app with the QueryClientProvider:

```typescript
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { Stack } from 'expo-router';
import { SafeAreaProvider } from 'react-native-safe-area-context';

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
  return (
    <QueryClientProvider client={queryClient}>
      {/* Your app content here */}
    </QueryClientProvider>
  );
}
```

## Step 3: Create Pokemon API Service

Create a new file `services/pokemon-api.ts` to handle Pokemon API calls:

```typescript
import { PokemonClient } from 'pokenode-ts';

// Create a singleton instance of the Pokemon client
export const PokeApiService = new PokemonClient();

// Export the service for use in hooks
export { PokeApiService };
```

## Step 4: Create Pokemon Hook

Create a new file `hooks/use-pokemon.ts` to implement the data fetching logic:

```typescript
import { useQuery } from '@tanstack/react-query';
import { PokeApiService } from '@/services/pokemon-api';

// Custom hook for fetching Pokemon list
export const usePokemonList = (limit: number = 20, offset: number = 0) => {
  return useQuery({
    queryKey: ['pokemon-list', limit, offset],
    queryFn: () => PokeApiService.listPokemons(limit, offset),
  });
};
```

## Step 5: Use the Hook in Your Component

Update your main Pokemon screen (`app/(tabs)/index.tsx`) to use the new hook:

```typescript
import { usePokemonList } from '@/hooks/use-pokemon';
import { View, Text, ActivityIndicator, FlatList, StyleSheet } from 'react-native';

export default function PokemonScreen() {
  const { data: pokemonList, isLoading, error } = usePokemonList(0, 150);

  if (isLoading) {
    return (
      <View style={styles.centerContainer}>
        <ActivityIndicator size="large" />
        <Text>Loading Pokemon...</Text>
      </View>
    );
  }

  if (error) {
    return (
      <View style={styles.centerContainer}>
        <Text>Error loading Pokemon: {error.message}</Text>
      </View>
    );
  }

  const renderPokemonItem = ({ item }: { item: any }) => (
    <View style={styles.pokemonItem}>
      <Text style={styles.pokemonName}>{item.name}</Text>
    </View>
  );

  return (
    <View style={styles.container}>
      <Text style={styles.title}>
        Pokemon List ({pokemonList?.length || 0})
      </Text>
      <FlatList
        data={pokemonList}
        renderItem={renderPokemonItem}
        keyExtractor={(item) => item.name}
        showsVerticalScrollIndicator={false}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 16,
  },
  centerContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 16,
  },
  pokemonItem: {
    padding: 12,
    backgroundColor: '#f0f0f0',
    marginBottom: 8,
    borderRadius: 8,
  },
  pokemonName: {
    fontSize: 16,
    fontWeight: '600',
    textTransform: 'capitalize',
  },
  pokemonId: {
    fontSize: 14,
    color: '#666',
    marginTop: 4,
  },
});
```

## Step 6: Add Pokemon ID with Select Function

Now let's enhance our hook to add Pokemon IDs to the data using the `select` function. Update your `hooks/use-pokemon.ts`:

```diff
import { useQuery } from '@tanstack/react-query';
+import { NamedAPIResource } from 'pokenode-ts';
import { PokeApiService } from '@/services/pokemon-api';

+// Type for Pokemon with ID
+export type PokemonWithId = NamedAPIResource & {
+  id: string;
+};

+// Helper function to extract Pokemon ID from URL
+export function getPokemonIdFromUrl(url: string): string | null {
+ if (!url) return null;
+
+ // Regex to match the ID in the URL pattern: /pokemon/{id}/
+ const match = url.match(/\/pokemon\/(\d+)\/?$/);
+ return match ? match[1] : null;
+}

+// Transform function to add ID to each Pokemon resource
+const mapWithResourceId = (resource: NamedAPIResource): PokemonWithId => {
+ const id = getPokemonIdFromUrl(resource.url) || '';
+ return {
+ id,
+    ...resource,
+ };
+};

## Step 7: Update Component to Use Pokemon IDs

Now update your component to display the Pokemon IDs:

```diff
  const renderPokemonItem = ({ item }: { item: any }) => (
    <View style={styles.pokemonItem}>
      <Text style={styles.pokemonName}>{item.name}</Text>
+     <Text style={styles.pokemonId}>ID: {item.id}</Text>
    </View>
  );
```

## Step 8: Test Your Implementation

1. Start your development server:
   ```bash
   npm run start
   ```

2. Navigate to your Pokemon screen and verify that:
   - Pokemon data loads successfully
   - Loading state is displayed while fetching
   - Error handling works (try with no internet connection)
   - Data is cached (navigate away and back to see instant loading)
   - Pokemon IDs are displayed alongside names

## Key Concepts Learned

- **QueryClient**: Central configuration for all queries
- **useQuery**: Hook for fetching and caching data
- **queryKey**: Unique identifier for caching
- **queryFn**: Function that returns a Promise
- **staleTime**: How long data is considered fresh
- **select**: Transform data before it's returned
- **Loading states**: Built-in loading, error, and success states

## Bonus

- Add error boundaries for better error handling
- Implement infinite scrolling with [`useInfiniteQuery`](https://tanstack.com/query/latest/docs/framework/react/reference/useInfiniteQuery)

## Troubleshooting

- **Import errors**: Make sure your TypeScript paths are configured correctly in `tsconfig.json`
- **Network errors**: Check your internet connection and API availability
- **Caching issues**: Clear your app data or restart the development server
- **Type errors**: Ensure all imports are correct and types are properly defined

## Resources

- [TanStack Query Documentation](https://tanstack.com/query/latest)
- [pokenode-ts Documentation](https://github.com/Gabb-c/pokenode-ts)
- [React Native Expo Router](https://expo.github.io/router/)
