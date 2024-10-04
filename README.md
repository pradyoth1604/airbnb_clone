import React from 'react';
import { COLORS, DB_ORG_LIST, STATUSNAMES } from './constants';
import { ifElse, isEmpty, isNullOrUndefined, orCheck } from 'hds-ui/utils';

// Write utility functions used by the RTv2 tool

/**
 * Represents a type that can be either of type T or null.
 *
 * @template T - The type that can be nullable.
 */
export type canBeNull<T> = T | null;

//Helper functions to get user info from token and api start
export const parseJWTToken = (token: string) => {
  if (!token) return null;
  const [, body] = token.split('.');
  if (!body || body === undefined) return null;
  try {
    return JSON.parse(atob(body));
  } catch (e) {
    return null;
  }
};

const getToken = () => {
  const apiToken: string = orCheck(
    localStorage.getItem('API_TOKEN'),
    '',
  ) as string;
  const claim = parseJWTToken(apiToken);
  return claim;
};

export const getUserStatus = (): boolean => {
  try {
    const token = getToken();
    const primary_tenant = token?.['primary_tenant'];
    const session_tenant = token?.['session_tenant'];

    if ((primary_tenant === '' && session_tenant === '') || !session_tenant)
      return true;
    if (primary_tenant === null && session_tenant === null) return true;
    if (!JSON.parse(DB_ORG_LIST as string).hasOwnProperty(session_tenant)) {
      return true;
    }
    return false;
  } catch (e) {
    console.error('an error occured', e);
    return false;
  }
};

export const formatDataForNameChange = (data: any) => {
  if (!data) return null;
  const formattedData = data?.map((template: any) => ({
    [template?.nameTemplate?.name]: template?.newName,
  }));
  return Object.assign({}, ...formattedData);
};
//Helper functions to get user info from token and api end

/**
 * Retrieves the value of a nested property from an object.
 * If the object is empty or the property is not found, it returns the default value.
 *
 * @param obj - The object to retrieve the property from.
 * @param property - An array of strings representing the nested property path.
 * @param defaultValue - The default value to return if the property is not found.
 * @returns The value of the nested property or the default value.
 */
export const getProperty = (
  obj: { [key: string]: any } = {},
  property: string[] = [],
  defaultValue: any = undefined,
) => {
  if (isEmpty(obj)) {
    return defaultValue;
  }
  let info: any = obj;
  for (const prop of property) {
    if (isEmpty(info)) {
      return defaultValue;
    } else {
      info = info[prop];
    }
  }
  return info;
};

/* Utility function to remove HTML tags from a given string.
 *
 * @param {string} html - The input string containing HTML tags.
 * @return {string} The input string with HTML tags removed.
 */
export const stripHtml = (html: string): string => {
  let tmp = document.createElement('DIV');
  tmp.innerHTML = html;
  return tmp.textContent || tmp.innerText || '';
};

export function formatRTCycleTitle(activeRTCycle: string) {
  if (activeRTCycle) {
    if (activeRTCycle === 'Next Cycle') return activeRTCycle;
    let rtCycle = activeRTCycle.split('-');
    rtCycle = rtCycle.map((cycle: string) => cycle.trim());
    const startMonth = rtCycle[0].slice(0, 3);
    const endMonth = rtCycle[1].slice(0, 3);
    const startYear = rtCycle[0].slice(-2);
    const endYear = rtCycle[1].slice(-2);
    return `${startMonth}‘${startYear} - ${endMonth}’${endYear}`;
  }
}

export const stripHtmlTags = (input: string): string => {
  const tempDiv = document.createElement('div');
  tempDiv.innerHTML = input;

  let points = Array.from(tempDiv.getElementsByTagName('p')).map(
    (p) => p.textContent?.trim() || '',
  );
  const listItems = Array.from(tempDiv.getElementsByTagName('li')).map(
    (p) => p.textContent?.trim() || '',
  );

  points = [...points, ...listItems];

  return points.filter((point) => point.length > 0).join('\n\n');
};

export function getStatusColor(
  status: string,
  managerAssessmentStatus: string = '',
) {
  status = status || 'NEW';
  const statusToColor: any = {
    N_A: COLORS.naColor,
    FINAL: COLORS.finalColor,
    NEW: COLORS.newColor,
    DRAFT: COLORS.draftColor,
    CLOSED: COLORS.closedColor,
  };

  if (managerAssessmentStatus === STATUSNAMES.final) {
    return statusToColor['CLOSED'];
  }

  return statusToColor[status];
}

export function getStatusMsg(
  status: string,
  isRT: boolean,
  managerAssessmentStatus: string = '',
) {
  status = status || 'NEW';
  const statusToMsg: any = {
    N_A: 'Not Enough Data',
    NEW: isRT ? 'Pending' : 'In progress',
    DRAFT: 'In progress',
    FINAL: 'Submitted',
    CLOSED: 'Closed',
  };

  if (managerAssessmentStatus === STATUSNAMES.final) {
    return statusToMsg['CLOSED'];
  }

  return statusToMsg[status];
}

export const getStatusInfo = (status: string) => {
  switch (status) {
    case 'NEW':
    case 'DRAFT':
      return {
        text: 'In Progress',
        color: 'statusInProgress',
        dotClass: 'inProgress',
      };
    case 'CLOSED':
      return {
        text: 'Closed',
        color: 'statusClosed',
        dotClass: 'closed',
      };
    case 'FINAL':
      return {
        text: 'Submitted',
        color: 'statusSubmitted',
        dotClass: 'final',
      };
    default:
      return {
        text: '',
        color: 'statusNA',
        dotClass: 'notEligible',
      };
  }
};

export const get1On1FeedbackPercentage = (data: any, month: any) => {
  let count = 0;
  data?.forEach((reportee: any) => {
    const reporteeMonth = reportee?.feedbackMeta?.filter((months: any) => {
      return months?.monthRange == month;
    });
    if (reporteeMonth[0]?.feedbackCount > 0) {
      count = count + 1;
    }
  });
  return count;
};
export const isOwnerCheck = (userId: string, assessment: any) => {
  return userId === assessment?.owner?.id;
};

export const isReadOnlyCheck = (assessment: any, isOwner: Boolean) => {
  return (
    assessment?.selfAssessment?.status === STATUSNAMES.FINAL ||
    !isOwner ||
    (!assessment?.cycle?.active && !assessment?.cycle?.isNextCycle)
  );
};

export const formatDrawerData = (questions: any, reviewOrSelfReview: any) => {
  let tempBands: string[] = [];
  const reviews = reviewOrSelfReview;
  questions = questions?.map((question: any) => {
    const review: any = reviews?.find(
      (e: any) => e.question.id === question.id,
    );
    if (question?.band) {
      tempBands.push(question?.band?.name);
    }
    question = { ...question, review: { choiceField: 0 } };
    question = assignReviewResponse(review, question);
    return question;
  });
  questions = questions?.toSorted((a: any, b: any) => a.order - b.order);
  tempBands = Array.from(new Set(tempBands));
  return {
    tempBands,
    questions,
  };
};

export const assignReviewResponse = (review: any, question: any) => {
  question['review']['choiceField'] = ifElse(
    review,
    review?.choiceField || review?.reviewerChoiceField,
    null,
  );
  question['review']['notAdequateResponse'] = ifElse(
    review,
    review?.notApplicable,
    null,
  );
  return question;
};

export const extractRtDecisions = (data: any) => {
  const rtDecisions = [];

  // Extract current rtDecision
  if (data?.rtDecision) {
    rtDecisions.push({
      ...data?.rtDecision,
      selfAssessment: {
        hierarchy: {
          cycle: {
            name: data?.hierarchy?.cycle?.name,
          },
        },
      },
    });
  }

  // Extract previous rtDecision
  if (data?.previousAssessment && data?.previousAssessment?.rtDecision) {
    rtDecisions.push({
      ...data?.previousAssessment?.rtDecision,
      selfAssessment: {
        hierarchy: {
          cycle: {
            name: data?.previousAssessment?.hierarchy?.cycle?.name,
          },
        },
      },
    });
  }

  return rtDecisions;
};

export const usePrevious = (value: any, defaultValue: any) => {
  const previousRef = React.useRef();

  React.useEffect(() => {
    previousRef.current = value;
  }, [value]);

  return ifElse(
    !isNullOrUndefined(defaultValue),
    orCheck(previousRef.current, defaultValue),
    previousRef.current,
  );
};



{

        title: '',

        render: (reviewer: ReviewerProps) => (

          

            {andCheck(

              getProperty(reviewer, ['managerAssessmentStatus'], '') ===

                STATUSNAMES.final,

                              style={{ color: COLORS.editColor }}

                onClick={() => {

                  navigate(`${APP_URL}/360/${reviewer.key}`, {

                    state: {

                      fromTable: true,

                    },

                  });

                }}

              />,

            )}

            {andCheck(

              getProperty(reviewer, ['status'], '') === STATUSNAMES.N_A,

              <>empty,

            )}

            {andCheck(

              orCheck(

                getProperty(reviewer, ['managerAssessmentStatus'], '') !==

                STATUSNAMES.final,

                getProperty(reviewer, ['status'], '') === STATUSNAMES.final,

              ),

                              style={{ color: COLORS.editColor }}

                onClick={() => {

                  navigate(`${APP_URL}/360/${reviewer.key}`, {

                    state: {

                      fromTable: true,

                    },

                  });

                }}

              />,

            )}

          

        ),

        key: 'edit',

        width: '5%',

      },

export function getStatusMsg(

  status: string,

  isRT: boolean,

  managerAssessmentStatus: string = '',

) {

  status = status || 'NEW';

  const statusToMsg: any = {

    N_A: 'Not Enough Data',

    NEW: isRT ? 'Pending' : 'In progress',

    DRAFT: 'In progress',

    FINAL: 'Submitted',

    CLOSED: 'Closed',

  };



  if (managerAssessmentStatus === STATUSNAMES.final) {

    return statusToMsg['CLOSED'];

  }



  return statusToMsg[status];

}

when i have status.NA i need nothing like eye or edit symbol instead if its pending or submitted which is final i need to see edit symbol
