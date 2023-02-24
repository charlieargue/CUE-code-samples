# State Pyramid

The Agency-codebase and team preferred a Global-Only approach to state management, whereby they didn't separate state into App and Server state, as well as put everything into a global Redux store.

By guiding and supporting the teams into a more "muscular" state management strategy, I introduced patterns for using React.Context effectively and avoiding over-rendering. 

I called this strategy a "State Pyramid", inspired by [Martin Fowler's Testing Pyramid](https://martinfowler.com/articles/practical-test-pyramid.html#TheTestPyramid) and Kent Dodds' [flow chart](https://kentcdodds.com/blog/state-colocation-will-make-your-react-app-faster) on where to put state.

Basically, move your state down and then "lift" it as necessary and only when necessary, and make it more sophisticated only when necessary:

* useState
* Lift it or co-locate it
* Component composition
* Compound components
* React.Context





## 👎 BEFORE: 

```js

```





- [ ] **React.Context:** 👎 BEFORE & 👍 AFTER: 

  - [ ] ```js 
    EVERY LITTLE THING!
    👎 BEFORE & (for real! it almost looks like a joke ;)
    // setUnreadMessages: (state) => {
            //     if (state.twilioModal) {
            //         return {
            //             ...state,
            //             twilioModal: {
            //                 ...state.twilioModal,
            //                 unreadMessages: true,
            //             },
            //         };
            //     }
            // },
      
      
      
      👍 AFTER: 
    
    case 'setUnreadMessages': {
      return { ...state, unreadMessages: true };
    }
    ```

