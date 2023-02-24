# Removing Code

*RTK-Q allowed us to remove thousands of lines of needless boilerplate code and actually **gain functionality** :*

* less over-rendering
* less network requests
* and less bugs!

*Moving the team away from the Agency's own cumbersome solution, and using Redux's RTK-Q instead, my refactoring effort eliminated the need to hand-write tedious and bug-prone code, such as the following example sample files, totaling in the thousands of lines of code and dozens of files:*



==Removed **SLICE** Files:==

```js 
import { slice as patientListingUISlice } from './patient-listing-ui/patient-listing-ui.slice';

// ...
    patientListing: ReturnType<typeof patientListingUISlice.getInitialState>;
    patientListing: patientListingUISlice.getInitialState(),
    patientListing: patientListingUISlice.reducer,

export const patientListingLocator = (s: PatientUIState) => s.patientListing;

```





==Removed **REDUCER** Files:==

```js 

// ...import { createSlice, PayloadAction } from '@reduxjs/toolkit/dist/query';
import { SearchPatient } from '../../models/patient';
import { patientListingUIInitialState, PatientListingUIState } from './patient-listing-ui.store';

export const slice = createSlice({
    name: 'patient/patient-listing-ui',
    initialState: patientListingUIInitialState,
    reducers: {
        setFilters: (state: PatientListingUIState, action: PayloadAction<SearchPatient | null>) => {
            state.filters = action.payload;
        },
        setPatientListingIds: (state: PatientListingUIState, action: PayloadAction<string[]>) => {
            state.patientListingIds = action.payload;
        },
        setPatientListingPagination: (state: PatientListingUIState, action: PayloadAction<number>) => {
            state.totalElements = action.payload;
        },
        setEncounterIds: (state: PatientListingUIState, action: PayloadAction<string[]>) => {
            state.encounterIds = action.payload;
        },
    },
});

// ...
export const actions = slice.actions;

```





==Removed **STORE** Files:==

```js 
import { SearchPatient } from '../../models/patient';

export interface PatientListingUIState {
    encounterIds?: string[];
    patientListingIds?: string[];
    filters?: SearchPatient;
    pageSize?: number;
    page?: number;
    totalElements?: number;
}

export const patientListingUIInitialState: PatientListingUIState = {
    filters: {
        id: [],
        name: [],
        phone: [],
    },
    pageSize: 6,
    page: 0,
    totalElements: 50,
};
// ...
```





==Removed **SELECTOR** Files:==

```js 
import { createSelector } from '@reduxjs/toolkit';
import { patientsSelectors } from '../../../fhir/store/patient-entity.selectors';
import { encountersSelectors } from '../../../fhir/store/encounter/encounter-entity.selectors';
import { patientToPatient } from '../../fhir-mappers/patient';
import { patientListingLocator } from '../patient-ui.store';
import { selectPatientUIState } from '../patient-ui.selectors';
import { PatientListingUIState } from './patient-listing-ui.store';
import { Patient, SearchPatient } from '../../models/patient';
import { isPresent } from 'ts-is-present';
import { IPatient, IEncounter } from '@ahryman40k/ts-fhir-types/lib/R4';

export const stateSelector = createSelector(selectPatientUIState, patientListingLocator);

export const filtersSelector = createSelector(
    stateSelector,
    (patientListingUIState: PatientListingUIState): SearchPatient | undefined => patientListingUIState.filters
);

export const selectedPatientListingIdsSelector = createSelector(
    stateSelector,
    (patientListingUIState: PatientListingUIState): string[] => patientListingUIState.patientListingIds || []
);

export const selectedEncounterIdsSelector = createSelector(
    stateSelector,
    (patientListingUIState: PatientListingUIState): string[] => patientListingUIState.encounterIds || []
);

export const selectedPatientListingSelector = createSelector(
    selectedPatientListingIdsSelector,
    selectedEncounterIdsSelector,
    patientsSelectors.entitySelectors.selectEntities,
    encountersSelectors.entitySelectors.selectEntities,
    (patientListingIds, encounterIds, entitiesPatients, entitiesEncounters): Patient[] => {
        const patientsEncounters = encounterIds.map((id: string) => entitiesEncounters[id]).filter(isPresent);

        return patientListingIds
            .map((id: string) => entitiesPatients[id])
            .filter(isPresent)
            .map((pat: IPatient) => {
                const myFirstEncounter = patientsEncounters.filter((enc) => {
                    const patId = pluckPatientId(enc);
                    return patId === pat.id;
                })[0];
                return patientToPatient(pat, null, null, myFirstEncounter);
            });
    }
);

export const selectPatientListingTotalElement = createSelector(
    stateSelector,
    (patientListingUIState: PatientListingUIState): number => patientListingUIState.totalElements
);

export const selectPatientListingPage = createSelector(
    stateSelector,
    (patientListingUIState: PatientListingUIState): number => patientListingUIState.page
);

export const selectPatientListingPageSize = createSelector(
    stateSelector,
    (patientListingUIState: PatientListingUIState): number => patientListingUIState.pageSize
);

// ...
```





