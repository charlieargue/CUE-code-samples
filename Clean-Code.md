# Clean Code

*Helped the team avoid over-engineered and obtuse code, hasty abstractions, and write maintainable, clean enterprise code by frequently rejecting agency-produced code. Agency engineers had a tendency to introduce unnecessarily complex functional programming code, which was impossible to maintain and onboard new team members with — even senior engineers found it difficult to work with.*

*For example, the following is a sample of code I stopped in favor of a more fundamental approach using `skipToken` directly, as seen in my other code samples.*



## 👎 BEFORE: 

```js
import { QueryDefinition, skipToken } from '@reduxjs/toolkit/dist/query';
import { QueryArgFrom, ResultTypeFrom } from '@reduxjs/toolkit/dist/query/endpointDefinitions';
import { UseQuery } from '@reduxjs/toolkit/dist/query/react/buildHooks';
import * as A from 'fp-ts/Array';

// ...
export type QueryResult<T> = QueryData<T> & QueryFetchStatus;

export const createBaseQueryResult = <T>(result: T, error: any): QueryData<T> => {
    return { currentData: result, error, isSuccess: !!result, isError: !!error };
};

type UseCombinedOrRTKQuery<Q> = Q extends QueryDefinition<any, any, any, any>
    ? UseQuery<Q>
    : Q extends CombinedQuery<any, any>
    ? Q
    : never;


// ...
type ResultTypeFromCombinedOrRTKQuery<Q> = Q extends QueryDefinition<any, any, any, any>
    ? ResultTypeFrom<Q>
    : Q extends CombinedQuery<any, infer R>
    ? R
    : never;


// ...
export const combineQueries3 = <
    P,
    A extends RTKQOrCombinedQuery,
    B extends RTKQOrCombinedQuery,
    C extends RTKQOrCombinedQuery,
    R
>(
    paramsFn: (p: typeof skipToken | P) => [QueryArg<A>, QueryArg<B>, QueryArg<C>],
    q1: UseCombinedOrRTKQuery<A>,
    q2: UseCombinedOrRTKQuery<B>,
    q3: UseCombinedOrRTKQuery<C>,
    combineFn: (
        arg0: ResultTypeFromCombinedOrRTKQuery<A>,
        arg1: ResultTypeFromCombinedOrRTKQuery<B>,
        arg2: ResultTypeFromCombinedOrRTKQuery<C>
    ) => R
): CombinedQuery<P, R> => {
    return combineQueriesAry([q1, q2, q3], paramsFn, ([a, b, c]) => combineFn(a, b, c));
};

export const combineQueries4 = <
    P,
    A extends RTKQOrCombinedQuery,
    B extends RTKQOrCombinedQuery,
    C extends RTKQOrCombinedQuery,
    D extends RTKQOrCombinedQuery,
    R
// ...

export const computeStuff = <T, R>(
    shouldSkipNext: boolean,
    f: (p: T) => { currentData: R; error: any },
    d: T
): { currentData: typeof skipToken | R; error: any } => {
    if (shouldSkipNext) return { currentData: skipToken, error: null };

    const fd = f(d);

    if (fd.error !== null) return { currentData: skipToken, error: fd.error };

    return fd;
};
        
// ... and then used in a RxJS fashion like this 
export const usePatientOrPatientsEncounters = combineQueries(
    ({ searchParams, singlePatientId }: { searchParams: Partial<PatientQuery>; singlePatientId?: string }) => {
        return [singlePatientId ?? skipToken, searchParams ?? skipToken];
    },
    usePatientEncounters,
    usePatientsEncounters,
    (singleResult, multipleResults) => {
        return singleResult ? [singleResult] : multipleResults;
    }
);
```

