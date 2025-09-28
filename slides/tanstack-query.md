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
