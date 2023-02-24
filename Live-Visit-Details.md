# Live Visit Details

*Sample code showing a more advanced **end-result**, after refactoring to RTK-Q, allowing the removal of several files of tedious and error-prone hand-written data fetching and caching logic.*





## 👍 AFTER: 

```tsx
import { R4 } from '@ahryman40k/ts-fhir-types';
import { skipToken } from '@reduxjs/toolkit/query/react';
import React from 'react';
import { useParams } from 'react-router-dom';
import { SearchCartridgeTestType } from '../../../cartridge-test-detail/api/builders';
import { useSearchTestByTestIdQuery } from '../../../cartridge-test-detail/api/cartridge-test-detail.endpoints';
import { unbundle } from '../../../fhir/be-api-common';
import { BundleResourceQuery } from '../../../fhir/query';
import {
    documentReferenceApi,
    DocumentReferenceQuery,
    encounterApi,
    observationApi,
    questionnaireApi,
    QuestionnaireQuery,
    questionnaireresponseApi,
    QuestionnaireresponseQuery,
} from '../../../fhir/query-store';
import { PatientVisitDetails } from '../../components/PatientVisitDetails/PatientVisitDetails';
import { usePatient } from '../../hooks/use-patient';
import { QuestionnaireIdTypes } from '../../models/questionnaire';
import { ObservationIdentifierIdTypes } from '../../models/test-details';

type qsBundleType = BundleResourceQuery<QuestionnaireQuery>[];

export const PatientVisitDetailsContainer: React.FC = () => {
    const { id: patientId, visitDetailsId: encounterId } = useParams<{ id: string; visitDetailsId?: string }>();
    const { currentData: patient, isLoading: isLoadingPatient, isFetching: isFetchingPatient } = usePatient(patientId);
    const {
        data: fhirEncounter,
        isLoading: isLoadingEncounter,
        isFetching: isFetchingEncounter,
    } = encounterApi.useFetchByIdQuery(encounterId);
    const qrsArgs: typeof skipToken | QuestionnaireresponseQuery = fhirEncounter?.id
        ? {
              patient: patientId,
              encounter: fhirEncounter.id,
          }
        : skipToken;
    const {
        data: fhirQRs,
        isLoading: isLoadingQRs,
        isFetching: isFetchingQRs,
    } = questionnaireresponseApi.useQueryQuery(qrsArgs);
    let qsArgs: typeof skipToken | qsBundleType = skipToken;
    if (fhirQRs?.length) {
        qsArgs = fhirQRs?.map(
            (q): BundleResourceQuery<QuestionnaireQuery> => ({
                kind: 'fetchById',
                id: q.questionnaire.split('/')[1],
            })
        );
    }
    const {
        data: fhirQsBundle,
        isLoading: isLoadingQs,
        isFetching: isFetchingQs,
    } = questionnaireApi.useBundleQuery(qsArgs);
    const fhirQs: R4.IQuestionnaire[] = fhirQsBundle?.map(unbundle) || undefined;
    const fhirQAntiviral = fhirQs?.find((q) => q.name === QuestionnaireIdTypes.ANTIVIRAL) || undefined;
    const fhirQRAntiviral = fhirQAntiviral
        ? fhirQRs?.find((qr) => qr.questionnaire === `Questionnaire/${fhirQAntiviral.id}`)
        : undefined;
    const fhirQHcp = fhirQs?.find((q) => q.name === QuestionnaireIdTypes.HCP_ANTIVIRAL) || undefined;
    const fhirQRHcp = fhirQHcp ? fhirQRs?.find((qr) => qr.questionnaire === `Questionnaire/${fhirQHcp.id}`) : undefined;
    const fhirQPost = fhirQs?.find((q) => q.name === QuestionnaireIdTypes.POST_CONSULTATION_SURVEY) || undefined;
    const fhirQRPost = fhirQPost
        ? fhirQRs?.find((qr) => qr.questionnaire === `Questionnaire/${fhirQPost.id}`)
        : undefined;
    const observationId: typeof skipToken | string =
        fhirEncounter?.reasonReference?.length > 0
            ? fhirEncounter.reasonReference[0].reference.substring('Observation/'.length)
            : skipToken;
    const {
        data: fhirO,
        isLoading: isLoadingO,
        isFetching: isFetchingO,
    } = observationApi.useFetchByIdQuery(observationId);
    const testId = fhirO?.identifier?.find(({ id }) => id === ObservationIdentifierIdTypes.TEST_ID)?.value || undefined;
    const testArgs: typeof skipToken | SearchCartridgeTestType = testId ? { testId: testId } : skipToken;
    const {
        currentData: test,
        isLoading: isLoadingTest,
        isFetching: isFetchingTest,
    } = useSearchTestByTestIdQuery(testArgs);
    const docRefsArgs: typeof skipToken | DocumentReferenceQuery = fhirQRAntiviral?.id
        ? { subject: `QuestionnaireResponse/${fhirQRAntiviral.id}` }
        : skipToken;
    const {
        data: fhirDocRefs,
        isLoading: isLoadingDocRefs,
        isFetching: isFetchingDocRefs,
    } = documentReferenceApi.useQueryQuery(docRefsArgs);

    const isLoading =
        isLoadingPatient ||
        isFetchingPatient ||
        isLoadingEncounter ||
        isFetchingEncounter ||
        isLoadingQRs ||
        isFetchingQRs ||
        isLoadingQs ||
        isFetchingQs ||
        isLoadingO ||
        isFetchingO ||
        isLoadingTest ||
        isFetchingTest ||
        isLoadingDocRefs ||
        isFetchingDocRefs;

    if (isLoading) {
        return <>Loading...</>;
    }

    return (
        <PatientVisitDetails
            patient={patient}
            encounter={fhirEncounter}
            qrAntiviral={fhirQRAntiviral}
            qAntiviral={fhirQAntiviral}
            qrHcp={fhirQRHcp}
            qrPost={fhirQRPost}
            observation={fhirO}
            testDetail={test}
            documentReferences={fhirDocRefs}
        />
    );
};		
```

