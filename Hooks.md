# Hooks

*Before refactoring to RTK-Q, first I introduced a re-usable hooks pattern that allowed all the tedious and error-prone Redux boilerplate code of selectors and useEffects and status derivations to be hidden inside re-usable hooks, and not duplicated all over the place and cluttering various components.*

*This also allowed the team to start using a standardized hooks consumption pattern that would be the same after we migrated to RTK-Q, minimizing the impact of the refactor.* 



:warning: NOTE: this is BEFORE refactoring to RTK-Q, so all old Redux code, just DRY-ed in a single place and made to follow a pattern in preparation for the refactor.

:warning: This kind of code used to be duplicated frequently in the codebase and would clutter component files, creating a lot of confusion and headache.

:warning: Moving this cumbersome code into hooks DRY-ed it and prepped for the RTK-Q refactor, by standardizing the consumption pattern. It became much easier for the team of new SDEs to work with it, as they were very familiar with hooks, but not with Redux.

```js 
export function useFetchPatientTestDetailsQuery(
    patientId: string,
    diagRepId: string = null,
    observationId: string = null
) {
    const { setDiagReport, setPatient, setPatientId, setTestDetailsId } = TestDetailsUIActions;
    const data = useSelector(selectedTestDetailsSelector);
    const status = useSelector(queryPendingTestDetailsSelector);
    const obzIds = useSelector(patientObservationIdsSelector);
    const obz = useSelector(patientObservationsSelector);

    useEffect(
        useAsyncEffect(async () => {
            if (!patientId || !observationId || !diagRepId) return;
            dispatch(PatientUICommonActions.setPatientId(patientId));
            dispatch(setPatientId(patientId));
            const fhirDiagReports = await dispatch(thunksDiagReport.query([`Patient/${patientId}`])).unwrap();
            const fhirObservation = await dispatch(thunksObservation.fetchById(observationId)).unwrap();
            const fhirPatient = await dispatch(thunksPatient.fetchById(patientId)).unwrap();
            const diagReport = fhirDiagReports.find((dr: R4.IDiagnosticReport) => dr.id === diagRepId);
            dispatch(setTestDetailsId((fhirObservation as any).id));
            dispatch(setDiagReport(diagReport));
            dispatch(setPatient(fhirPatient as unknown as R4.IPatient));
        }),
        [dispatch, observationId, patientId, diagRepId]
    );

    const isLoading = status === AsyncStatus.Pending || status === AsyncStatus.Void;
    const isUninitialized = status === AsyncStatus.Void;
    const isError = status === AsyncStatus.Rejected;
    const isSuccess = status === AsyncStatus.Fulfilled;

    return {
        data,
        isUninitialized,
        isLoading,
        isError,
        isSuccess,
    };
}

// consumpation / usage:
// -------------------------
// const { testDetails, isUninitialized, isLoading, isError, isSuccess } =
//  useFetchPatientTestDetailsQuery('123abc', '4d5e', 'o212312');

```

