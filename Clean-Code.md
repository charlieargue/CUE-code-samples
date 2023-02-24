# Clean Code

*Helped the team avoid over-engineered and unnecessary code, hasty abstractions, and write maintainable enterprise code by frequently rejecting agency-produced code. Agency engineers had a tendency to introduce unnecessarily complex functional programming code, which was difficult to maintain and onboard new team members. Stopped this in favor of a more fundamental approach using `skipToken` directly, as seen in other code samples here.*





## 👎 BEFORE: 

```js
import { QueryDefinition, skipToken } from '@reduxjs/toolkit/dist/query';
import { QueryArgFrom, ResultTypeFrom } from '@reduxjs/toolkit/dist/query/endpointDefinitions';
import { UseQuery } from '@reduxjs/toolkit/dist/query/react/buildHooks';
import * as A from 'fp-ts/Array';

export type QueryFetchStatus = {
    isLoading: boolean;
    isFetching: boolean;
    isUninitialized: boolean;
};

export type QueryData<T> = {
    currentData?: T;
    error?: any;
    isSuccess: boolean;
    isError: boolean;
};

export type QueryResult<T> = QueryData<T> & QueryFetchStatus;

export const createBaseQueryResult = <T>(result: T, error: any): QueryData<T> => {
    return { currentData: result, error, isSuccess: !!result, isError: !!error };
};

type CombinedQuery<T, R> = (p: T | typeof skipToken) => QueryResult<R>;

type UseCombinedOrRTKQuery<Q> = Q extends QueryDefinition<any, any, any, any>
    ? UseQuery<Q>
    : Q extends CombinedQuery<any, any>
    ? Q
    : never;

type QueryArgFromCombinedOrRTKQuery<Q> = Q extends QueryDefinition<any, any, any, any>
    ? QueryArgFrom<Q>
    : Q extends CombinedQuery<infer T, any>
    ? T
    : never;

type QueryArg<Q> = typeof skipToken | QueryArgFromCombinedOrRTKQuery<Q>;

type ResultTypeFromCombinedOrRTKQuery<Q> = Q extends QueryDefinition<any, any, any, any>
    ? ResultTypeFrom<Q>
    : Q extends CombinedQuery<any, infer R>
    ? R
    : never;

export type RTKQOrCombinedQuery = CombinedQuery<any, any> | QueryDefinition<any, any, any, any>;

export const combineQueries = <P, A extends RTKQOrCombinedQuery, B extends RTKQOrCombinedQuery, R>(
    paramsFn: (p: typeof skipToken | P) => [QueryArg<A>, QueryArg<B>],
    q1: UseCombinedOrRTKQuery<A>,
    q2: UseCombinedOrRTKQuery<B>,
    combineFn: (arg0: ResultTypeFromCombinedOrRTKQuery<A>, arg1: ResultTypeFromCombinedOrRTKQuery<B>) => R
): CombinedQuery<P, R> => {
    return combineQueriesAry([q1, q2], paramsFn, ([a, b]) => combineFn(a, b));
};

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
>(
    paramsFn: (p: typeof skipToken | P) => [QueryArg<A>, QueryArg<B>, QueryArg<C>, QueryArg<D>],
    q1: UseCombinedOrRTKQuery<A>,
    q2: UseCombinedOrRTKQuery<B>,
    q3: UseCombinedOrRTKQuery<C>,
    q4: UseCombinedOrRTKQuery<D>,
    combineFn: (
        arg0: ResultTypeFromCombinedOrRTKQuery<A>,
        arg1: ResultTypeFromCombinedOrRTKQuery<B>,
        arg2: ResultTypeFromCombinedOrRTKQuery<C>,
        arg3: ResultTypeFromCombinedOrRTKQuery<D>
    ) => R
): CombinedQuery<P, R> => {
    return combineQueriesAry([q1, q2, q3, q4], paramsFn, ([a, b, c, d]) => combineFn(a, b, c, d));
};

const createCaptureResult =
    <T, R>(combineFn: (p: T) => R) =>
    (a: T): { currentData: R; error: any } => {
        try {
            const result = combineFn(a);
            return { currentData: result, error: null };
        } catch (e) {
            return { currentData: null, error: e };
        }
    };

const createCaptureResult2 =
    <T1, T2, R>(t: (arg0: T1, arg1: T2) => R) =>
    (p1: T1, p2: T2) =>
        createCaptureResult(([p1, p2]: [T1, T2]) => t(p1, p2))([p1, p2]);

export const combineQueriesAry =
    <Params extends readonly any[], R>(
        queries: UseCombinedOrRTKQuery<any>[],
        params: (...args: Params) => any[],
        combineFn: (arg0: any[]) => R
    ) =>
    (...args: Params) => {
        const queryParams = params(...args);

        const queryResults = A.zip(queries, queryParams).map(([query, queryParam]) => query(queryParam));

        const captureResult = createCaptureResult(combineFn);

        const combineResult =
            queryResults.every((qr) => !qr.isError) && queryResults.some((qr) => !qr.isUninitialized)
                ? captureResult(queryResults.map((qr) => qr.currentData))
                : { currentData: null, error: null };

        const error =
            combineResult.error || (queryResults.some((qr) => qr.isError) ? queryResults.map((qr) => qr.error) : null);

        return {
            ...createBaseQueryResult(combineResult.currentData, error),
            isLoading: queryResults.some((qr) => qr.isLoading),
            isFetching: queryResults.some((qr) => qr.isFetching),
            isUninitialized: queryResults.some((qr) => qr.isUninitialized),
        };
    };

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

export const combineQueriesSequential = <A extends RTKQOrCombinedQuery, B extends RTKQOrCombinedQuery, R>(
    q1: UseCombinedOrRTKQuery<A>,
    t1: (p1: ResultTypeFromCombinedOrRTKQuery<A>) => QueryArgFromCombinedOrRTKQuery<B>,
    q2: UseCombinedOrRTKQuery<B>,
    t2: (p1: ResultTypeFromCombinedOrRTKQuery<A>, p2: ResultTypeFromCombinedOrRTKQuery<B>) => R
): CombinedQuery<QueryArgFromCombinedOrRTKQuery<A>, R> => {
    const t1Executor = createCaptureResult(t1);
    const t2Executor = createCaptureResult2(t2);
    return (p1: QueryArg<A>): QueryResult<R> => {
        const q1r = q1(p1);
        const shouldSkipNext = q1r.isFetching || q1r.isUninitialized || q1r.isError;
        const p2 = computeStuff(shouldSkipNext, t1Executor, q1r.currentData);
        const q2r = q2(p2.currentData);
        const finalResult = t2Executor(q1r.currentData, q2r.currentData);
        return {
            isError: q1r.isError || p2.error !== null || q2r.isError || finalResult.error !== null,
            isSuccess: q1r.isSuccess && p2.error === null && q2r.isSuccess && finalResult.error === null,
            isLoading: q1r.isLoading || q2r.isLoading,
            isFetching: q1r.isFetching || q2r.isFetching,
            isUninitialized: q1r.isUninitialized && q2r.isUninitialized,
            currentData: finalResult.currentData,
        };
    };
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

