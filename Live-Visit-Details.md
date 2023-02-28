# Live Visit Details

*<u>Sample code</u> showing a more advanced **end-result**, after refactoring to RTK-Q, allowing the removal once again of several **files** of tedious and error-prone hand-written data fetching and caching logic.*





## 👍 AFTER: 

```tsx
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
    const fhirQAntiviral = fhirQs?.find((q) => q.name === QuestionnaireIdTypes.SANITIZED) || undefined;
    const fhirQRAntiviral = fhirQAntiviral
        ? fhirQRs?.find((qr) => qr.questionnaire === `Questionnaire/${fhirQAntiviral.id}`)
        : undefined;
    const fhirQHcp = fhirQs?.find((q) => q.name === QuestionnaireIdTypes.NA) || undefined;
    const fhirQRHcp = fhirQHcp ? fhirQRs?.find((qr) => qr.questionnaire === `Questionnaire/${fhirQHcp.id}`) : undefined;
    const fhirQPost = fhirQs?.find((q) => q.name === QuestionnaireIdTypes.TBD) || undefined;
    const fhirQRPost = fhirQPost
        ? fhirQRs?.find((qr) => qr.questionnaire === `Questionnaire/${fhirQPost.id}`)
        : undefined;
    const observationId: typeof skipToken | string =
        fhirEncounter?.reasonReference?.length > 0
            ? fhirEncounter.reasonReference[0].reference.substring('Lorem/'.length)
            : skipToken;
    const {
        data: fhirO,
        isLoading: isLoadingO,
        isFetching: isFetchingO,
    } = observationApi.useFetchByIdQuery(observationId);
    const testId = fhirO?.identifier?.find(({ id }) => id === ObservationIdentifierIdTypes.LOREM)?.value || undefined;
    const testArgs: typeof skipToken | SearchCartridgeTestType = testId ? { testId: testId } : skipToken;
    const {
        currentData: test,
        isLoading: isLoadingTest,
        isFetching: isFetchingTest,
    } = useSearchTestByTestIdQuery(testArgs);
    const docRefsArgs: typeof skipToken | DocumentReferenceQuery = fhirQRAntiviral?.id
        ? { subject: `Ipsum/${fhirQRAntiviral.id}` }
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

