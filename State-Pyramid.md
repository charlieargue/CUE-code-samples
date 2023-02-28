# State Pyramid

The agency's team preferred a "Global-Only" approach to state management, whereby they didn't separate state into App and Server state, and put everything into a global Redux store.

By guiding and supporting the teams into a more "muscular" state management strategy, I introduced a "State Pyramid" pattern, inspired by [Martin Fowler's Testing Pyramid](https://martinfowler.com/articles/practical-test-pyramid.html#TheTestPyramid) and Kent Dodds' [flow chart](https://kentcdodds.com/blog/state-colocation-will-make-your-react-app-faster) on where to put state.

It advocated for:

1. Separating server state from application state
2. Using RTK-Q for handling all server state
3. And using various minimal and native React state management solutions for application state



In a nutshell, I re-trained the team to be suspicious of state, to always be trying to have as little of it as possible (and letting go of the predecessor's preference for the opposite), and to put state where it should be, depending on requirements, namely:

1. Local `useState`
2. Lifted or co-located
3. Component composition
4. Compound components
5. React.Context



This approach allowed for, amongst other things:

- Removal of dozens of cumbersome and hard-to-reason-about `useEffects`
- Often entire removal of state management code (eg. replaced by local state, RTK-Q, or was redundant in the first place)
- More readable code that was easier to maintain
- Virtually bug-free code (contrasting the bug-filled code the agency team produced)
- A more responsive and snappy application and UI/UX (and less network requests)



## 👎 Example before `React.context`:

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





# 👍 After `React.Context`

```js 
case 'setUnreadMessages': {
  return { ...state, unreadMessages: true };
}
```

