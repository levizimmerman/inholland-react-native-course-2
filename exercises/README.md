# React Native Course Exercises - Pokedex App

This README contains all the exercise information for building a Pokedex app using React Native. Each exercise builds upon the previous one, teaching you the fundamentals of React Native development with Expo, TypeScript, and modern state management while creating a functional Pokémon application.

## Table of Contents

1. [Exercise 1: Adding TanStack Query](./exercise-1-adding-tanstack.md) - Setting up data fetching and state management
2. [Exercise 2: Fetching Pokemon Data by Name](./exercise-2-fetching-pokemon.md) - Creating detail pages and navigation
3. [Exercise 3: Favorite Pokemon with Local Database](./exercise-3-favorite-pokemon.md) - Implementing favorites with SQLite

---

## General Submission Guidelines

For all exercises:

1. **Create a GitHub repository** for your project
2. **Commit your code** after completing each exercise
3. **Write clear commit messages** describing what you implemented
4. **Test your app** ideally on both iOS and Android.
5. **Push your code to GitHub** and share the repository with [me via Teams](https://teams.microsoft.com/l/chat/48:notes/conversations?context=%7B%22contextType%22%3A%22chat%22%7D).

## Helpful Commands

- `npx expo start` - Start development server
- `npx expo start --clear` - Start with cleared cache
- `npx expo doctor` - Check for common issues
- `npx expo install [package]` - Install Expo-compatible packages
- `npm run start` - Alternative start command (used in exercises)

### Exercise-Specific Commands

**Exercise 1 (TanStack Query):**
- `npm install @tanstack/react-query` - Install TanStack Query
- `npm install pokenode-ts` - Install Pokemon API client

**Exercise 3 (Database):**
- `npx expo install expo-sqlite` - Install SQLite for local database

> **📚 Reference:** [Expo CLI Commands](https://docs.expo.dev/more/expo-cli/)

## Additional Resources

### Documentation Links
- [Expo Documentation](https://docs.expo.dev/) - Complete Expo guide
- [React Native Documentation](https://reactnative.dev/) - Official React Native docs
- [Expo Router Documentation](https://docs.expo.dev/router/introduction/) - Navigation guide
- [React Native Components](https://reactnative.dev/docs/components-and-apis) - All available components

### Exercise-Specific Documentation
- [TanStack Query Documentation](https://tanstack.com/query/latest) - Data fetching and state management
- [Expo SQLite Documentation](https://docs.expo.dev/versions/latest/sdk/sqlite/) - Local database operations
- [pokenode-ts Documentation](https://github.com/Gabb-c/pokenode-ts) - Pokemon API client
- [Expo Haptics](https://docs.expo.dev/versions/latest/sdk/haptics/) - Device haptic feedback

### Learning Resources
- [Expo Snack](https://snack.expo.dev/) - Online code editor for testing
- [Expo Go App](https://expo.dev/client) - Test your apps on your phone
- [React Native Tutorial](https://reactnative.dev/docs/tutorial) - Official tutorial

## Troubleshooting

### General Issues
If you encounter issues:
1. Make sure you have Node.js installed (version 18+)
2. Try `npx expo start --clear` to clear cache
3. Restart your terminal and try again
4. Check the [Expo documentation](https://docs.expo.dev/)
5. Ask your instructor for help!

### Exercise-Specific Troubleshooting

**Exercise 1 (TanStack Query):**
- **Import errors**: Make sure your TypeScript paths are configured correctly in `tsconfig.json`
- **Network errors**: Check your internet connection and API availability
- **Caching issues**: Clear your app data or restart the development server
- **Type errors**: Ensure all imports are correct and types are properly defined

**Exercise 2 (Navigation):**
- **Route not found**: Make sure your `[name].tsx` file is in the correct directory
- **Image not loading**: Check your internet connection and image URL
- **Navigation issues**: Verify the route path matches your file structure

**Exercise 3 (Database):**
- **Database not initialized**: Make sure to call `initDatabase()` in your app layout
- **SQLite errors**: Check that all database operations are properly awaited
- **Query not updating**: Ensure you're invalidating the correct query keys
- **Import errors**: Verify all file paths and imports are correct

Good luck with your Pokedex app! 🚀