# Exercise 2: Fetching Pokemon Data by Name

In this exercise, you'll learn how to fetch individual Pokemon data by name and display it on a detail page. You'll also create a Pokemon image component to display detailed illustrations.

## Prerequisites

- Completed Exercise 1 (TanStack Query setup)
- Basic understanding of React Native navigation
- Familiarity with TypeScript

## Step 1: Add Pokemon by Name Hook

First, let's add a new hook to fetch Pokemon data by name. Update your `hooks/use-pokemon.ts`:

```typescript
// Add this new hook to your existing file
export const usePokemonByName = (name: string) => {
  return useQuery({
    queryKey: ['pokemon', name],
    queryFn: () => PokeApiService.getPokemonByName(name),
    enabled: !!name, // Only run query if name is provided
    staleTime: 10 * 60 * 1000, // 10 minutes
  });
};
```

## Step 2: Create Pokemon Detail Page

Create a new file `app/pokemon/[name].tsx` for the Pokemon detail screen:

```typescript
import { usePokemonByName } from '@/hooks/use-pokemon';
import { useLocalSearchParams } from 'expo-router';
import { ActivityIndicator, ScrollView, StyleSheet, Text, View } from 'react-native';
import { SafeAreaView } from 'react-native-safe-area-context';

export default function PokemonDetailScreen() {
  const { name } = useLocalSearchParams();
  const { data: pokemon, isLoading, error } = usePokemonByName(name as string);

  if (isLoading) {
    return (
      <SafeAreaView style={styles.container}>
        <View style={styles.loadingContainer}>
          <ActivityIndicator size="large" color="#5631E8" />
          <Text style={styles.loadingText}>Loading Pokémon...</Text>
        </View>
      </SafeAreaView>
    );
  }

  if (error || !pokemon) {
    return (
      <SafeAreaView style={styles.container}>
        <View style={styles.errorContainer}>
          <Text style={styles.errorText}>Pokémon not found</Text>
        </View>
      </SafeAreaView>
    );
  }

  return (
    <SafeAreaView style={styles.container}>
      <ScrollView style={styles.scrollView}>
        <View style={styles.header}>
          <Text style={styles.pokemonName}>
            {pokemon.name.charAt(0).toUpperCase() + pokemon.name.slice(1)}
          </Text>
          <Text style={styles.pokemonId}>#{pokemon.id.toString().padStart(3, '0')}</Text>
        </View>
        
        {/* Pokemon Image will go here */}
        <View style={styles.imageContainer}>
          <Text style={styles.placeholderText}>Pokemon Image</Text>
        </View>
        
        <View style={styles.detailsContainer}>
          <Text style={styles.sectionTitle}>Types</Text>
          <View style={styles.typesContainer}>
            {pokemon.types.map((typeInfo, index) => (
              <View key={index} style={styles.typeBadge}>
                <Text style={styles.typeText}>
                  {typeInfo.type.name.charAt(0).toUpperCase() + 
                   typeInfo.type.name.slice(1)}
                </Text>
              </View>
            ))}
          </View>
        </View>
      </ScrollView>
    </SafeAreaView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f0f8ff',
  },
  scrollView: {
    flex: 1,
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
  errorContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  errorText: {
    fontSize: 18,
    color: '#666',
  },
  header: {
    alignItems: 'center',
    paddingVertical: 20,
    paddingHorizontal: 16,
  },
  pokemonName: {
    fontSize: 32,
    fontWeight: 'bold',
    color: '#0E0940',
    textTransform: 'capitalize',
  },
  pokemonId: {
    fontSize: 18,
    color: '#666',
    marginTop: 4,
  },
  imageContainer: {
    alignItems: 'center',
    paddingVertical: 20,
    backgroundColor: '#fff',
    marginHorizontal: 16,
    borderRadius: 12,
    marginBottom: 16,
  },
  placeholderText: {
    fontSize: 16,
    color: '#999',
  },
  detailsContainer: {
    padding: 16,
    backgroundColor: '#fff',
    marginHorizontal: 16,
    borderRadius: 12,
  },
  sectionTitle: {
    fontSize: 20,
    fontWeight: 'bold',
    color: '#0E0940',
    marginBottom: 12,
  },
  typesContainer: {
    flexDirection: 'row',
    flexWrap: 'wrap',
    gap: 8,
  },
  typeBadge: {
    backgroundColor: '#5631E8',
    paddingHorizontal: 16,
    paddingVertical: 8,
    borderRadius: 20,
  },
  typeText: {
    color: '#fff',
    fontWeight: 'bold',
    textTransform: 'capitalize',
  },
});
```

## Step 3: Create Pokemon Image Component

Create a new file `components/ui/pokemon-image.tsx`:

```typescript
import React from 'react';
import { Image, StyleSheet, View } from 'react-native';

interface PokemonImageProps {
  id: string | number;
  size?: number;
}

export function PokemonImage({ id, size = 200 }: PokemonImageProps) {
  const imageUrl = `https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/official-artwork/${id}.png`;
  
  return (
    <View style={[styles.container, { width: size, height: size }]}>
      <Image
        source={{ uri: imageUrl }}
        style={[styles.image, { width: size, height: size }]}
        resizeMode="contain"
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    justifyContent: 'center',
    alignItems: 'center',
  },
  image: {
    backgroundColor: 'transparent',
  },
});
```

## Step 4: Update Pokemon Detail Page with Image

Now update your `app/pokemon/[name].tsx` to use the PokemonImage component:

```diff
import { usePokemonByName } from '@/hooks/use-pokemon';
+import { PokemonImage } from '@/components/ui/pokemon-image';
import { useLocalSearchParams } from 'expo-router';
// ... other imports

export default function PokemonDetailScreen() {
  // ... existing code

  return (
    <SafeAreaView style={styles.container}>
      <ScrollView style={styles.scrollView}>
        <View style={styles.header}>
          <Text style={styles.pokemonName}>
            {pokemon.name.charAt(0).toUpperCase() + pokemon.name.slice(1)}
          </Text>
          <Text style={styles.pokemonId}>#{pokemon.id.toString().padStart(3, '0')}</Text>
        </View>
        
-        {/* Pokemon Image will go here */}
-        <View style={styles.imageContainer}>
-          <Text style={styles.placeholderText}>Pokemon Image</Text>
-        </View>
+        <View style={styles.imageContainer}>
+          <PokemonImage id={pokemon.id} size={200} />
+        </View>
        
        // ... rest of the component
      </ScrollView>
    </SafeAreaView>
  );
}
```

## Step 5: Add Navigation from Pokemon List

Update your Pokemon list component to navigate to the detail page. In your `components/ui/pokemon-list.tsx` or wherever you display Pokemon cards:

```typescript
import { router } from 'expo-router';

// In your Pokemon card component or list item
const handlePokemonPress = (pokemonName: string) => {
  router.push(`/pokemon/${pokemonName}`);
};

// Use this in your onPress handler
<Pressable onPress={() => handlePokemonPress(pokemon.name)}>
  {/* Your Pokemon card content */}
</Pressable>
```

## Step 6: Test Your Implementation

1. Start your development server:
   ```bash
   npm run start
   ```

2. Navigate to your Pokemon list and tap on any Pokemon

3. Verify that:
   - The detail page loads with the correct Pokemon name
   - The Pokemon image displays correctly
   - Loading states work properly
   - Error handling works (try with an invalid Pokemon name)
   - Navigation works smoothly

## Key Concepts Learned

- **Dynamic Routes**: Using `[name].tsx` for parameterized routes
- **useLocalSearchParams**: Accessing route parameters in Expo Router
- **Conditional Queries**: Using `enabled` to control when queries run
- **Image Loading**: Loading images from external URLs
- **Navigation**: Programmatic navigation with `router.push()`

## Bonus Features

- Add Pokemon stats (height, weight, abilities)
- Implement a back button
- Add loading skeletons for better UX

## Troubleshooting

- **Route not found**: Make sure your `[name].tsx` file is in the correct directory
- **Image not loading**: Check your internet connection and image URL
- **Navigation issues**: Verify the route path matches your file structure
- **Type errors**: Ensure all imports are correct and types are properly defined

## Resources

- [Expo Router Documentation](https://expo.github.io/router/)
- [React Native Image Component](https://reactnative.dev/docs/image)
- [TanStack Query useQuery](https://tanstack.com/query/latest/docs/framework/react/reference/useQuery)
