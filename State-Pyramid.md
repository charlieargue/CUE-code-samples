# State Pyramid

The Agency-codebase and team preferred a Global-Only approach to state management, whereby they didn't separate state into App and Server state, as well as put everything into a global Redux store.

By guiding and supporting the teams into a more "muscular" state management strategy, I introduced patterns for using React.Context effectively and avoiding over-rendering. 

I called this strategy a "State Pyramid", inspired by [Martin Fowler's Testing Pyramid](https://martinfowler.com/articles/practical-test-pyramid.html#TheTestPyramid) and Kent Dodds' [flow chart](https://kentcdodds.com/blog/state-colocation-will-make-your-react-app-faster) on where to put state.

Basically, move your state down and then "lift" it as necessary and only when necessary, and make it more sophisticated only when necessary:

1. useState
2. Lift it or co-locate it
3. Component composition
4. Compound components
5. React.Context



This approach allowed for:

- Removal of dozens of no longer necessary and cumbersome `useEffects`
- Often entire removal of state management code (eg. Replaced by local state, RTK-Q, or was redundant in the first place)
- More readable code that was easier to maintain



## 👎 Before:

```js
setUnreadMessages: (state) => {
            if (state.twilioModal) {
                return {
                    ...state,
                    twilioModal: {
                        ...state.twilioModal,
                        unreadMessages: true,
                    },
                };
            }
        },
```





# 👍 After: `using React.Context`

```js 
case 'setUnreadMessages': {
  return { ...state, unreadMessages: true };
}
```

