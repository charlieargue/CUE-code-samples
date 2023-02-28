# Account Profile

*<u>Sample code</u> showing removal of over a thousand lines of Redux boilerplate and redundant state management code that was bug-prone and difficult to maintain, even for senior engineers.*



## 👎 BEFORE: 

* lots of selectors and everything in REDUX
* state over-complicated and stored twice (in Formik state and again in Redux)
* lots of `useEffects` making it hard to reason about this component, esp. for new React engineers
* **PLUS many Redux boilerplate <u>files</u> , totaling more than a THOUSAND of lines of code:**
  1. actions (AsyncThunk)

  2. api fetchers functions

  3. selectors

  4. stores

  5. reducers


```js
// 👎 BEFORE: 
const AccountProfileContainer: FC = () => {
    const dispatch = useAppDispatch();
    const profile = useSelector(selectProfileData);
    const isProfilePending = useSelector(selectIsProfileStatusPending);
    const isUpdateProfilePending = useSelector(selectIsUpdateProfileStatusPending);
    const isUpdateProfilePicturePending = useSelector(selectIsUpdateProfilePictureStatusPending);
    const pracititionerFHIRResource = useSelector(selectPractitionerFHIRResource);
    const usStates = useSelector(selectStatesAbbreviation);
    const isUSStatesPending = useSelector(selectIsStatesStatusPending);
    const i18nConfig = useSelector(selectConfigData);
    const [formData, setFormData] = useState<AccountProfileFormValues>({
        firstName: '',
        middleName: '',
        lastName: '',
        email: '',
        type: AuthTypes.UserType.Home,
        phoneType: null,
        phoneNumber: '',
        zipCode: '',
        city: '',
        birthDate: '',
        stateOfResidence: '',
        prefix: '',
        suffix: '',
        picture: '',
        fax: '',
        address: '',
    });

    useEffect(() => {
        dispatch(fetchProfile({}));
        dispatch(fetchStates());
        dispatch(fetchConfig());
    }, []);

    useEffect(() => {
        if (!profile) return;
        if (profile.practitionerId) {
            dispatch(practitionerThunks.fetchById(profile.practitionerId));
        }
        setFormData({
            ...formData,
            ...profileToAccountProfileFormValues(profile),
        });
    }, [profile]);

    useEffect(() => {
        if (!pracititionerFHIRResource) return;
        setFormData({
            ...formData,
            ...practitionerToAccountProfileFormValues(pracititionerFHIRResource),
        });
    }, [pracititionerFHIRResource]);
  
  // .... PLUS DOZENS of SELECTORS  
  export const selectIsTokenStatusPending = createSelector(
    selectTokenStatus,
    (status: AsyncStatus): boolean => status === AsyncStatus.Pending
  );

  export const selectUpdateProfile = createSelector(
      selectAccountState,
      (account: AccountState): AsyncState<null> => account.updateProfile
  );

  export const selectUpdateProfileStatus = createSelector(
      selectUpdateProfile,
      (updateProfile: AsyncState<null>): AsyncStatus => updateProfile.status
  );

  export const selectIsUpdateProfileStatusPending = createSelector(
      selectUpdateProfileStatus,
      (status: AsyncStatus): boolean => status === AsyncStatus.Pending
  );

  export const selectUpdateProfilePicture = createSelector(
      selectAccountState,
      (account: AccountState): AsyncState<null> => account.updateProfilePicture
  );

  
  // .... FETCHERS and ASYNC THUNKS
  export const updateProfilePicture = createAsyncThunk(
    'account/updateProfilePicture',
    withError(
        async (payload: { email: string; picture: File }, { dispatch }): Promise<void> => {
            const { email, picture } = payload;
            await updateProfilePictureRequest({ email, picture });
            notification.success({ message: 'Profile picture updated.' });
            dispatch(fetchProfile({}));
        },
        { customErrorMessage: 'An error occurred while updating the profile picture' }
    ),
    {
        condition: (payload, { getState }) =>
            checkActionCondition({ state: getState(), status: selectUpdateProfilePictureStatus(getState()) }),
    }
);

  
    // .... REDUCERS
    builder
        .addCase(updateProfile.pending, (state) => ({
            ...state,
            updateProfile: {
                ...state.updateProfile,
                status: AsyncStatus.Pending,
            },
        }))
        .addCase(updateProfile.fulfilled, (state) => ({
            ...state,
            updateProfile: {
                ...state.updateProfile,
                status: AsyncStatus.Fulfilled,
            },
        }))
        .addCase(updateProfile.rejected, (state) => ({
            ...state,
            updateProfile: {
                ...state.updateProfile,
                status: AsyncStatus.Rejected,
            },
        }));
    builder
        .addCase(updateProfilePicture.pending, (state) => ({
            ...state,
            updateProfilePicture: {
                ...state.updateProfilePicture,
                status: AsyncStatus.Pending,
            },
        }))
        .addCase(updateProfilePicture.fulfilled, (state) => ({
            ...state,
            updateProfilePicture: {
                ...state.updateProfilePicture,
                status: AsyncStatus.Fulfilled,
            },
        }))
        .addCase(updateProfilePicture.rejected, (state) => ({
            ...state,
            updateProfilePicture: {
                ...state.updateProfilePicture,
                status: AsyncStatus.Rejected,
            },
        }));

		// ... etc. over a thousand lines of this kind of boileraplate...

```



## 👍 AFTER: 

* RTK-Q for data-fetching and cache management
* no selectors and no redundant state, Formik handles everything
* no `useEffects` 
* all Redux boilerplate files **deleted**

```js 
// 👍 AFTER: 
const AccountProfileContainer: FC = () => {
    const dispatch = useAppDispatch();
    const [triggerAddPractitioner] = practitionerApi.useAddMutation();
    const { data: stateList, isLoading: isLoadingStates, isFetching: isFetchingStates } = useFetchStatesQuery();
    const usStates = (stateList || []).map(({ abbreviation }) => abbreviation);
    const { data: profile, isLoading: isLoadingProfile } = useGetMeQuery();
    const { data: i18nConfig, isLoading: isLoadingConfig, isFetching: isFetchingConfig } = useFetchConfigQuery();
    let pracArgs: typeof skipToken | string = skipToken;
    if (profile?.practitionerId) {
        pracArgs = profile?.practitionerId;
    }
    const {
        data: fhirPractitioner,
        isLoading: isLoadingPractitionerFHIR,
        isFetching: isFetchingPractitionerFHIR,
    } = practitionerApi.useFetchByIdQuery(pracArgs);

    const isLoading =
        isLoadingStates ||
        isFetchingStates ||
        isLoadingProfile ||
        isLoadingPractitionerFHIR ||
        isFetchingPractitionerFHIR ||
        isLoadingConfig ||
        isFetchingConfig;

    if (isLoading) {
        return <>Loading...</>;
    }
    return <AccountProfile .......

    // ... and an RTK-Q builder file
    builder.mutation<AuthTypes.AuthUser, Partial<AuthTypes.AuthUser>>({
        query: (resource) => ({
            url: `/me`,
            method: 'put',
            data: resource,
        }),
        invalidatesTags: (result, error, { id }) => {
            return [{ type: 'User', id: 'ME' }];
        },
    });
```

