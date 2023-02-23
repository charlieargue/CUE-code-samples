# Hooks

*Before refactoring to RTK-Q, first I introduced a re-usable hooks pattern that allowed all the tedious and error-prone Redux boilerplate code of selectors and useEffects and status derivations to be hidden inside re-usable hooks, so that is was not duplicated everywhere and cluttering various components.*

*This also allowed the team to start using a standardized hooks consumption pattern that would be the same when we migrated to RTK-Q, minimizing the impact of the refactor.* 



:warning: NOTE: this is BEFORE refactoring to RTK-Q, so all old Redux code, just DRY-ed in a single place and made to follow a pattern in preparation for the refactor.

:warning: This kind of code used to be duplicated everywhere in the codebase and would clutter component files, creating a lot of confusion and headache. 

:warning: Moving that cumbersome code into hooks DRY-ed it and prepped for the RTK-Q refactor, standardized the consumption pattern, and allowed junior devs to easily consume it and be protected from all the implementation details while I refactored from RTK to RTK-Q.

```js 
import { R4 } from '@ahryman40k/ts-fhir-types';
import { useEffect } from 'react';
import { useSelector } from 'react-redux';
import { useAsyncEffect } from '../../../lib/use-async-effect';
import { thunks as thunksDiagReport } from '../../../modules/fhir/store/diagnostic-report/diagnostic-report-entity-adapter.slice';
import { thunks as thunksObservation } from '../../../modules/fhir/store/observation/observation-adapter.slice';
import { thunks as thunksPatient } from '../../../modules/fhir/store/patient-entity-adapter.slice';
import {
    patientObservationIdsSelector,
    patientObservationsSelector,
} from '../../../modules/patient/store/common/selectors';
import { actions as PatientUICommonActions } from '../../../modules/patient/store/common/slice';
import {
    queryPendingSelector as queryPendingTestDetailsSelector,
    selectedTestDetailsSelector,
} from '../../../modules/patient/store/test-details-ui/test-details-ui.selectors';
import { actions as TestDetailsUIActions } from '../../../modules/patient/store/test-details-ui/test-details-ui.slice';
import { dispatch } from '../../../store';
import { AsyncStatus } from '../../../types/AsyncStatus';

// ##################################################################################
// USAGE:
//          `const { testDetails, isUninitialized, isLoading, isError, isSuccess } = useFetchPatientTestDetailsQuery('123abc', '4d5e', 'o212312');`
//
// ##################################################################################
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

            // set Ids
            dispatch(PatientUICommonActions.setPatientId(patientId));
            dispatch(setPatientId(patientId));

            // fhir requests
            const fhirDiagReports = await dispatch(thunksDiagReport.query([`Patient/${patientId}`])).unwrap();
            const fhirObservation = await dispatch(thunksObservation.fetchById(observationId)).unwrap();
            const fhirPatient = await dispatch(thunksPatient.fetchById(patientId)).unwrap();
            const diagReport = fhirDiagReports.find((dr: R4.IDiagnosticReport) => dr.id === diagRepId);

            // set IDs
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
```

