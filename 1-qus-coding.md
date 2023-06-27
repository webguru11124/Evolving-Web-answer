
# question 1

We're interested in understanding how you code and what drives your decisions while coding. To understand this, could you send us a short snippet of code (100 to 500 lines) that you wrote with your explanations of what it does or solves and why you decided to code it this way. You can put them in a Github Gist, Github repository, PasteBin or a Google Doc.

## what does it?

This code snippet appears to be a React component called `MemberModal`. It is a react component with typescript show member modal which has create and update view feature.

Let's break it down:

1. The code imports various dependencies and components needed for the `MemberModal` component.

2. The `MemberModal` component receives props such as `selectedLocations`, `selectedProfessions`, `name`, `id`, `onAudienceSubmited`, and `onClose`.

3. Inside the component, it uses hooks such as `useDispatch`, `useForm`, and custom hooks like `useGetProfessionsQuery` and `useCreateAudienceMutation` to handle state, form validation, and API requests.

4. The component renders a modal that contains a form for creating or updating an audience. It includes input fields for the audience name, location dropdown, professions dropdown, and displays a list of audience users.

5. The component defines event handlers such as `onSubmit`, `saveAudience`, `editAudience`, and `onAudienceSubmitedFromArticles` to handle form submissions and API requests.

6. The component also includes UI elements such as buttons, icons, and a spinner to indicate loading or processing states.

Overall, the `MemberModal` component appears to be a reusable modal component that handles audience creation or update functionality. It utilizes various React hooks, form validation, and API requests to provide a user-friendly interface for managing audiences.

## motivation to make code like this

The code follows certain patterns and practices to achieve readability, maintainability, and modularity. Here are some reasons why the code might be structured in this style:

1. **Component-based architecture:** The code is organized into a modular structure where each component handles a specific functionality. This allows for better code organization, reusability, and separation of concerns.

2. **Functional components and hooks:** The code utilizes functional components and React hooks, which are recommended approaches in modern React development. Functional components are simpler, easier to test, and promote code reuse. Hooks provide a way to manage state and side effects without using class components.

3. **Separation of concerns:** The code separates different concerns into separate modules and components. For example, data fetching and mutations are handled by separate custom hooks, form validation is abstracted using a resolver, and UI components are isolated into their own modules. This separation makes the code easier to understand, test, and maintain.

4. **Declarative and expressive syntax:** The code uses declarative syntax and expressive naming conventions. This makes the code more readable and self-explanatory. For example, the use of descriptive variable and function names helps to understand their purpose and functionality.

5. **Reusability and modularity:** The code promotes reusability by utilizing components, hooks, and utility functions. This allows for easier code sharing and reduces duplication. For example, the `LocationDropdown` and `ProfessionsDropdown` components can be reused in other parts of the application.

6. **Consistency and best practices:** The code follows established best practices and coding conventions. This promotes consistency across the codebase, improves code readability, and makes it easier for other developers to understand and contribute to the project.

Overall, the chosen code style and structure aim to improve code quality, maintainability, and developer productivity. However, it's important to note that coding style can vary depending on personal preferences, team conventions, and project requirements.

```

import { cx } from '@emotion/css';
import { zodResolver } from '@hookform/resolvers/zod';
import useGetUsersQuery from 'app/api/audiences/hooks/useGetUsersQuery';
import { Audience } from 'app/api/audiences/types';
import { Profession, Location } from 'app/api/auth/types';
import useGetProfessionsQuery from 'app/api/professions/hooks/useProfessionsQuery';
import { Input2, Modal, Spinner } from 'app/components';
import {
  useAudiencesTranslation,
  useCommonTranslation,
} from 'app/internationalization/hooks';
import useUpdateAudienceMutation from 'app/pages/Audience/hooks/useUpdateAudienceMutation';
import { actions } from 'app/store/editor';
import { actions as modalActions } from 'app/store/modal';
import { Information } from 'iconsax-react';
import { useForm } from 'react-hook-form';
import { useDispatch } from 'react-redux';
import CloseLineIcon from 'remixicon-react/CloseLineIcon';

import useCreateAudienceMutation from '../../../hooks/useCreateAudienceQuery';
import LocationDropdown from '../../LocationTree/LocationDropdown';
import ProfessionsDropdown from '../../ProfessionsDropdown';
import { FormFields, schema } from '../memberFormSchema';

import AudienceUsers from './AudienceUsers';

export interface MemberModalProps {
  selectedLocations: Location[];
  selectedProfessions: Profession[];
  id?: number;
  name?: string;
  onAudienceSubmited?: (result: Audience | null) => void;
  onClose?: VoidFunction;
}

const MemberModal = ({
  selectedLocations,
  selectedProfessions,
  name,
  id,
  onAudienceSubmited,
  onClose,
}: MemberModalProps) => {
  const dispatch = useDispatch();
  const { t } = useAudiencesTranslation();
  const { t: tc } = useCommonTranslation();

  const { data: professions, isLoading: isLoadingProfessions } =
    useGetProfessionsQuery();
  const { mutate: createAudience, isCreating } = useCreateAudienceMutation();
  const { mutate: updateAudience, isUpdating } = useUpdateAudienceMutation();

  const { register, handleSubmit, control, formState, watch } = useForm({
    defaultValues: {
      name: name ?? '',
      locations: selectedLocations.map((l) => l.id),
      professions: selectedProfessions.map((p) => p.id),
    },
    resolver: zodResolver(schema),
    mode: 'onChange',
  });

  const { data: users, isLoading: isLoadingUsers } = useGetUsersQuery({
    locations: watch('locations'),
    professions: watch('professions'),
  });

  const close = () => {
    if (!onClose) {
      dispatch(modalActions.hideModal());
      return;
    }

    onClose();
  };

  const onSubmit = (data: FormFields) => {
    const dataToSubmit = {
      ...data,
      name: {
        en: data.name,
      },
    };
    if (id) {
      editAudience(dataToSubmit);
      return;
    }
    saveAudience(dataToSubmit);
  };

  const saveAudience = (data: {
    name: {
      en: string;
    };
    locations: number[];
    professions: number[];
  }) => {
    createAudience(data, {
      onSuccess: (result) => {
        if (!onAudienceSubmited) {
          onAudienceSubmitedFromArticles(result.data.data);
          return;
        }
        onAudienceSubmited(result.data.data);
        close();
      },
      onError: () => close(),
    });
  };

  const editAudience = (data: {
    name: {
      en: string;
    };
    locations: number[];
    professions: number[];
  }) => {
    if (!id) return;

    updateAudience(
      { data, id },
      {
        onSuccess: (result) => {
          onAudienceSubmited?.(result.data.data);
          close();
        },
        onError: (e) => close(),
      }
    );
  };

  const onAudienceSubmitedFromArticles = (audience: Audience) => {
    if (!audience) return;
    close();
    dispatch(
      actions.addAudience({
        id: audience.id,
        name: audience.name,
        members: users?.length ?? 0,
      })
    );
  };

  return (
    <Modal onClose={close} className="max-w-[488px] h-[714px] py-4">
      <div className="flex flex-col h-full">
        <div className="flex relative">
          <div className="flex flex-col mr-[50px]">
            <span className="font-bold text-grayscale-primary">
              {t('Create Audience')}
            </span>
            <span className="text-xs text-grayscale-secondary">
              {t(
                'Audience is based on locations and professions you select below. Once created it will visible in your audience list.'
              )}
            </span>
          </div>
          <button
            className="h-10 w-10 flex justify-center items-center rounded bg-white shadow-atobi text-grayscale-secondary absolute right-0"
            onClick={close}
          >
            <CloseLineIcon />
          </button>
        </div>

        <form
          className="flex flex-col h-full mt-5"
          onSubmit={handleSubmit(onSubmit)}
        >
          <div className="mb-4">
            <label
              className="block font-bold text-sm text-grayscale-primary mb-2"
              htmlFor="audience"
            >
              {t('Audience Name')}
              <span className="text-error">*</span>
            </label>
            <Input2
              register={register}
              name="name"
              placeholder="Type name for audience"
              isSearch={false}
            />
          </div>
          <div className="mb-4">
            <span className="block font-bold text-sm text-grayscale-primary mb-2">
              {t('Filters')}
            </span>
            <div className="flex justify-between">
              <div className="flex flex-col w-[212px] relative">
                <LocationDropdown
                  control={control}
                  preSelectedLocations={selectedLocations}
                />
                <span className="text-xs text-grayscale-secondary">
                  <span className="text-error">*</span>
                  {tc('required')}
                </span>
              </div>
              <div className="w-[212px] relative">
                <ProfessionsDropdown
                  selectedProfessions={selectedProfessions}
                  professions={professions}
                  control={control}
                />
              </div>
            </div>
          </div>
          <div className="flex items-center">
            <Information className="text-focus" />
            <span className="text-xs text-grayscale-secondary ml-3">
              {t(
                'New employees will automatically join the audience based on their location & profession'
              )}
            </span>
          </div>
          <AudienceUsers isLoading={isLoadingUsers} users={users} />
          <div className="flex justify-center w-full mt-auto">
            {(isCreating || isUpdating) && <Spinner />}
            {!isCreating && !isUpdating && (
              <button
                className={cx('w-[248px] h-12 rounded-xl text-sm', {
                  'bg-focus text-white hover:bg-hover-primary':
                    formState.isValid,
                  'bg-gray-light text-grayscale-secondary': !formState.isValid,
                })}
              >
                {id ? t('Update Audience') : t('Save Audience')}
              </button>
            )}
          </div>
        </form>
      </div>
    </Modal>
  );
};

export default MemberModal;

```
