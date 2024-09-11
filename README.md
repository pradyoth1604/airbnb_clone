# airbnb_clone


import React from 'react';
import ReviewerAssessmentTemplate from '../../templates/ReviewerAssessmentTemplate/ReviewerAssessmentTemplate';
import { useParams } from 'react-router-dom';
import { ENGINEERING_REVIEW_FEEDBACK_QUERY } from '../../utils/queries';
import { useQueryHook } from '../../utils/client';
import ReviewerFeedback from '../ReviewerFeedback';

const ReviewerAssessmentPage: React.FC = () => {
  const { id } = useParams();
  const {
    loading: feedbackLoading,
    error: feedbackError,
    data: feedbackData,
  } = useQueryHook(ENGINEERING_REVIEW_FEEDBACK_QUERY, {
    variables: {
      reviewerAssessmentId: id,
    },
    fetchPolicy: 'network-only', 
  });


  return (
    <div className="container">
      <ReviewerFeedback feedbackData={feedbackData} />
      <ReviewerAssessmentTemplate
        feedbackData={feedbackData}
        feedbackError={feedbackError}
        feedbackLoading={feedbackLoading}
        id={id}
      />
    </div>
  );
};

export default ReviewerAssessmentPage;

import { ArrowLeftIcon } from 'hds-ui/icons';
import { Spin } from 'hds-ui/components';
import ProfileCardWholeTemplate from '../templates/ProfileCardWholeTemplate/ProfileCardWholeTemplate';
import { REVIEW_SUMMARY_FOR_MANAGER, TENANT_REVIEW_LIST } from '../utils/queries';
import { Link, useParams } from 'react-router-dom';
import RtFinalFeedBackContainer from './RtFinalFeedBackContainer/RtFinalFeedbackContainer';
import { APP_URL } from '../utils/constants';
import ProfileCard from '../templates/ProfileCardWholeTemplate/ProfileCard';

interface ReviewerFeedbackProps {
    feedbackData: any;
}

const ReviewerFeedback: React.FC<ReviewerFeedbackProps> = ({feedbackData}) => {

if (!feedbackData) {
    return <div>Loading....</div>;
  }
  return (
    <>
      {' '}
      <div className="manager-feedback-page-container">
        <Link to={`${APP_URL}/manager-assessment`}>
          <ArrowLeftIcon color="black" size={30} strokeWidth={1.5} />
        </Link>
        <span
          style={{ paddingLeft: '1rem', fontSize: '20px', fontWeight: '100' }}>
          Reviewer's Assessment
        </span>
      </div>
      <ProfileCard data={feedbackData} flagToHideManagerAssessmentData={true} />
      {/* <ProfileCardWholeTemplate data={data} /> */}
      {/* <RtFinalFeedBackContainer /> */}
    </>
  );
};

export default ReviewerFeedback;

import './ProfileCardWholeTemplate.scss';
import ProfileCardWithDropdown from '../../modules/ProfileCardWithDropdown/ProfileCardWithDropdown';
import ProfilePreviousRTCard from '../../molecules/ProfilePreviousRTCard/ProfilePreviousRTCard';
import ProfileOfManagerAndCoach from '../../molecules/ProfileOfManagerAndCoach/ProfileOfManagerAndCoach';
import ProfilePreviousExperience from '../../molecules/ProfilePreviousExperience/ProfilePreviousExperience';
import ProfileTimelineButton from '../../atoms/ProfileTimelineButton';
import ProfileUtilisationScore from '../../molecules/ProfileUtilisationScore/ProfileUtilisationScore';
import RTQuestionsDrawer from '../RTQuestionsDrawer/RTQuestionsDrawer';
import { useState } from 'react';
import { getProperty } from '../../utils';
import ProfileCardAssess from '../../modules/ProfileCardWithDropdown/ProfileCardAssess';

interface ProfileCardProps {
  data: any;
  flagToHideManagerAssessmentData: boolean;
}
const ProfileCard: React.FC<ProfileCardProps> = ({
  data,
  flagToHideManagerAssessmentData
}) => {
//   const { selfAssessment: apiSelfAssessment } = data || {};
//   const selfAssessment = apiSelfAssessment?.[0] || {};
//   const { hierarchy, timeSpentOnCurrentBand } = selfAssessment;
//   const owner = selfAssessment?.hierarchy?.owner;
//   const profile = owner?.profile;
//   const selfAssessmentId = selfAssessment?.id;
  const [isDrawerVisible, setIsDrawerVisible] = useState(false);
  const handleRTClick = () => {
    setIsDrawerVisible(true);
  };
  // Assuming `data` is the object containing the JSON structure
const { reviews } = data || {};
const review = reviews?.[0] || {};
const { selfAssessment: apiSelfAssessment } = review || {};
const selfAssessment = apiSelfAssessment || {};
const { hierarchy, timeSpentOnCurrentBand } = selfAssessment || {};
const owner = hierarchy?.owner || {};
const profile = owner?.profile || {};
const selfAssessmentId = review?.id || null;

  if (!data) {
    return <div>No feedback data available.</div>;
  }
  console.log("Data>>>>>>>>>>>", data);
  return (
    <>
      <div className="profile-card-main" data-testid="profile-template">
      <ProfileCardAssess
      flagToHideManagerAssessmentData = {flagToHideManagerAssessmentData}
          profile={profile}
          id={selfAssessment}
          email={owner?.email}
          contributors={{
            reviewers: getProperty(selfAssessment, ['allReviewers'], []),
            reviewers360: getProperty(selfAssessment, ['reviewers360'], []),
          }}
          manager={getProperty(selfAssessment, ['manager'], {})}
        />
        {/* <ProfileCardWithDropdown
          profile={profile}
          id={selfAssessment?.id}
          email={owner?.email}
          contributors={{
            reviewers: getProperty(selfAssessment, ['allReviewers'], []),
            reviewers360: getProperty(selfAssessment, ['reviewers360'], []),
          }}
          manager={getProperty(hierarchy, ['manager'], {})}
        /> */}
        <ProfilePreviousRTCard
          previousAssessment={selfAssessment?.previousAssessment}
        />
        <div className="prev-exp-utilisation">
          <ProfilePreviousExperience
            profile={profile}
            hierarchy={hierarchy}
            timeSpentOnCurrentBand={timeSpentOnCurrentBand}
          />
         {!flagToHideManagerAssessmentData && <ProfileUtilisationScore
            userEmail={hierarchy?.owner?.email}
            cycleId={hierarchy?.cycle?.id}
          />}
        </div>
        <ProfileOfManagerAndCoach
          manager={hierarchy?.manager}
          coach={hierarchy?.coach}
          flagToHideManagerAssessmentData = {flagToHideManagerAssessmentData}
        />
        <div className="profile-btn-margin">
          <ProfileTimelineButton
            className={'career-timeline-btn'}
            title={'Career Timeline'}
            selfAssessmentId={selfAssessmentId}
          />
          <br />
          <ProfileTimelineButton
            className={'expectations-margin'}
            title={'RT Questions'}
            handleRTClick={handleRTClick}
          />
          <br />
          {!flagToHideManagerAssessmentData && <ProfileTimelineButton
            id={owner?.id}
            className={'expectations-margin'}
            title={'Manager 1 on 1 Summary'}
          />}
        </div>
      </div>
      {setIsDrawerVisible && (
        <RTQuestionsDrawer
          setIsDrawerVisible={setIsDrawerVisible}
          isDrawerVisible={isDrawerVisible}
          assessmentId={selfAssessmentId}
        />
      )}
    </>
  );
};

export default ProfileCard;

import './ProfileCardWholeTemplate.scss';
import ProfileCardWithDropdown from '../../modules/ProfileCardWithDropdown/ProfileCardWithDropdown';
import ProfilePreviousRTCard from '../../molecules/ProfilePreviousRTCard/ProfilePreviousRTCard';
import ProfileOfManagerAndCoach from '../../molecules/ProfileOfManagerAndCoach/ProfileOfManagerAndCoach';
import ProfilePreviousExperience from '../../molecules/ProfilePreviousExperience/ProfilePreviousExperience';
import ProfileTimelineButton from '../../atoms/ProfileTimelineButton';
import ProfileUtilisationScore from '../../molecules/ProfileUtilisationScore/ProfileUtilisationScore';
import RTQuestionsDrawer from '../RTQuestionsDrawer/RTQuestionsDrawer';
import { useState } from 'react';
import { getProperty } from '../../utils';

interface ProfileCardWholeTemplateProps {
  data: any;
}
const ProfileCardWholeTemplate: React.FC<ProfileCardWholeTemplateProps> = ({
  data,
}) => {
  console.log("manager",data)
  const { selfAssessment: apiSelfAssessment } = data || {};
  const selfAssessment = apiSelfAssessment?.[0] || {};
  const { hierarchy, timeSpentOnCurrentBand } = selfAssessment;
  const owner = selfAssessment?.hierarchy?.owner;
  const profile = owner?.profile;
  const selfAssessmentId = selfAssessment?.id;
  const [isDrawerVisible, setIsDrawerVisible] = useState(false);
  const handleRTClick = () => {
    setIsDrawerVisible(true);
  };

  return (
    <>
      <div className="profile-card-main" data-testid="profile-template">
        <ProfileCardWithDropdown
          profile={profile}
          id={selfAssessment?.id}
          email={owner?.email}
          contributors={{
            reviewers: getProperty(selfAssessment, ['allReviewers'], []),
            reviewers360: getProperty(selfAssessment, ['reviewers360'], []),
          }}
          manager={getProperty(hierarchy, ['manager'], {})}
        />
        <ProfilePreviousRTCard
          previousAssessment={selfAssessment?.previousAssessment}
        />
        <div className="prev-exp-utilisation">
          <ProfilePreviousExperience
            profile={profile}
            hierarchy={hierarchy}
            timeSpentOnCurrentBand={timeSpentOnCurrentBand}
          />
          <ProfileUtilisationScore
            userEmail={hierarchy?.owner?.email}
            cycleId={hierarchy?.cycle?.id}
          />
        </div>
        <ProfileOfManagerAndCoach
          manager={hierarchy?.manager}
          coach={hierarchy?.coach}
        />
        <div className="profile-btn-margin">
          <ProfileTimelineButton
            className={'career-timeline-btn'}
            title={'Career Timeline'}
            selfAssessmentId={selfAssessmentId}
          />
          <br />
          <ProfileTimelineButton
            className={'expectations-margin'}
            title={'RT Questions'}
            handleRTClick={handleRTClick}
          />
          <br />
          <ProfileTimelineButton
            id={owner?.id}
            className={'expectations-margin'}
            title={'Manager 1 on 1 Summary'}
          />
        </div>
      </div>
      {setIsDrawerVisible && (
        <RTQuestionsDrawer
          setIsDrawerVisible={setIsDrawerVisible}
          isDrawerVisible={isDrawerVisible}
          assessmentId={selfAssessmentId}
        />
      )}
    </>
  );
};

export default ProfileCardWholeTemplate;


import UserCard from 'hds-ui/components/UserCard';
import './ProfileCardWithDropdown.scss';
import ProfileDropdownReportees from '../../molecules/ProfileDropdownReportees/ProfileDropdownReportees';
import ProfileReviewersAnd360 from '../../molecules/ProfileReviewersAnd360';
import DetailedProfile from '../../molecules/DetailedProfile/DetailedProfile';
import { dashboardClient, useQueryHook } from '../../utils/client';
import {
  GET_ACE_CERTIFICATIONS,
  GET_SKILLS,
  INTERVIEWS_COUNT,
} from '../../utils/queries';
import { Popover } from 'hds-ui/components';
import ProfileDropDownRI from '../../molecules/ProfileDropdownReportees/ProfileDropDownRI';

interface ProfileCardAssessProps {
  flagToHideManagerAssessmentData : boolean
  profile: any;
  id: any;
  email: string;
  contributors: {
    reviewers: Array<any>;
    reviewers360: Array<any>;
  };
  manager: any;
}
const ProfileCardAssess: React.FC<ProfileCardAssessProps> = ({
    flagToHideManagerAssessmentData,
  profile,
  id,
  email,
  contributors,
  manager,
}) => {
  const { name, designation, profilePic } = profile || '';
  const { track, band} = profile || {};
  const { data } = useQueryHook(GET_ACE_CERTIFICATIONS, {
    variables: { email: email, selfAssessmentId: id },
  });
  const { data: skillsData } = useQueryHook(GET_SKILLS, {
    variables: {
      conditions: {
        email: email,
      },
    },
    client: dashboardClient,
  });

  const { data: hireData } = useQueryHook(INTERVIEWS_COUNT, {
    variables: { email: email, selfAssessmentId: id },
  });
  const interviewsCount = hireData?.hireInterviewsCount?.[0]?.interviewCount;
  const usersCertifiedCertificationList =
    data?.aceCertifications?.result?.filter((obj: any) => {
      return obj?.userCertificate;
    });
  const userSkills = skillsData?.getUser?.detailedProfile?.userSkills;
  return (
    <div className="profile-user-container" data-testid="detailed-profile">
      <div className="main-profile-user-card">
        <Popover
          placement="rightTop"
          content={
            <DetailedProfile
              name={name}
              track={band?.name}
              designation={designation}
              usersCertifiedCertificationList={usersCertifiedCertificationList}
              userSkills={userSkills}
              interviewsCount={interviewsCount}
            />
          }>
          <div>
            <UserCard
              profilePic={profilePic}
              description={`Track - ${track?.name}`}
              band={band?.name}
              title={name}
              subTitle={designation}
              loading={false}
              imgSize="lg"
              imgHoverZoom={false}
            />
          </div>
        </Popover>
        <ProfileDropDownRI />
        {/* <ProfileDropdownReportees /> */}
      </div>
      {!flagToHideManagerAssessmentData && <ProfileReviewersAnd360 info={contributors} manager={manager} />}
    </div>
  );
};

export default ProfileCardAssess;

import UserCard from 'hds-ui/components/UserCard';
import './ProfileCardWithDropdown.scss';
import ProfileDropdownReportees from '../../molecules/ProfileDropdownReportees/ProfileDropdownReportees';
import ProfileReviewersAnd360 from '../../molecules/ProfileReviewersAnd360';
import DetailedProfile from '../../molecules/DetailedProfile/DetailedProfile';
import { dashboardClient, useQueryHook } from '../../utils/client';
import {
  GET_ACE_CERTIFICATIONS,
  GET_SKILLS,
  INTERVIEWS_COUNT,
} from '../../utils/queries';
import { Popover } from 'hds-ui/components';

interface ProfileCardWithDropdown {
  profile: any;
  id: any;
  email: string;
  contributors: {
    reviewers: Array<any>;
    reviewers360: Array<any>;
  };
  manager: any;
}
const ProfileCardWithDropdown: React.FC<ProfileCardWithDropdown> = ({
  profile,
  id,
  email,
  contributors,
  manager,
}) => {
  const { name, designation, profilePic } = profile || '';
  const { track, band } = profile || {};
  const { data } = useQueryHook(GET_ACE_CERTIFICATIONS, {
    variables: { email: email, selfAssessmentId: id },
  });
  const { data: skillsData } = useQueryHook(GET_SKILLS, {
    variables: {
      conditions: {
        email: email,
      },
    },
    client: dashboardClient,
  });

  const { data: hireData } = useQueryHook(INTERVIEWS_COUNT, {
    variables: { email: email, selfAssessmentId: id },
  });
  const interviewsCount = hireData?.hireInterviewsCount?.[0]?.interviewCount;
  const usersCertifiedCertificationList =
    data?.aceCertifications?.result?.filter((obj: any) => {
      return obj?.userCertificate;
    });
  const userSkills = skillsData?.getUser?.detailedProfile?.userSkills;
  return (
    <div className="profile-user-container" data-testid="detailed-profile">
      <div className="main-profile-user-card">
        <Popover
          placement="rightTop"
          content={
            <DetailedProfile
              name={name}
              track={track?.name}
              designation={designation}
              usersCertifiedCertificationList={usersCertifiedCertificationList}
              userSkills={userSkills}
              interviewsCount={interviewsCount}
            />
          }>
          <div>
            <UserCard
              profilePic={profilePic}
              description={`Track - ${track?.name}`}
              band={band?.name}
              title={name}
              subTitle={designation}
              loading={false}
              imgSize="lg"
              imgHoverZoom={false}
            />
          </div>
        </Popover>
        <ProfileDropdownReportees />
      </div>
      <ProfileReviewersAnd360 info={contributors} manager={manager} />
    </div>
  );
};

export default ProfileCardWithDropdown;


import { MenuProps, UserCard } from 'hds-ui/components';
import { TEAM_LIST } from '../../utils/queries';
import { useQueryHook } from '../../utils/client';
import exclamation from '../../assets/exclamation.svg';
import check from '../../assets/check.svg';
import { Link } from 'react-router-dom';
import './ProfileDropdownReportees.scss';
import ProfileDropdownReporteesAtom from '../../atoms/ProfileDropdownReporteesAtom';
import { APP_URL } from '../../utils/constants';

const ProfileDropdownReportees = () => {
  const { data, error } = useQueryHook(TEAM_LIST);
  if (error) {
    return <></>;
  }
  console.log("ProfileDropdownReportees",data)
  const currentURL = window.location.href;
  const TeamData = data?.myTeamAssessment?.result;
  const json = TeamData?.map((data: any, index: number) => ({
    key: (index + 1).toString(),
    label: (
      <Link
        to={
          currentURL?.includes('final')
            ? `${APP_URL}/manager-final-feedback/${data?.selfAssessment?.id}`
            : `${APP_URL}/manager-feedback-form/${data?.selfAssessment?.id}`
        }>
        <div style={{ display: 'flex' }}>
          {data?.selfAssessment?.managerAssessment?.status === 'FINAL' ? (
            <img src={check} className="dropdown-status" />
          ) : (
            <img src={exclamation} className="dropdown-status" />
          )}
          <UserCard
            profilePic={data?.owner?.profile?.profilePic}
            band={data?.owner?.profile?.band?.name}
            title={data?.owner?.profile?.name}
            subTitle={data?.owner?.profile?.designation}
            loading={false}
            imgSize="md"
            imgHoverZoom={false}
          />
        </div>
      </Link>
    ),
    disabled:
      !data?.selfAssessment?.id ||
      (currentURL?.includes('final') &&
        data?.selfAssessment?.rtDecision?.status != 'FINAL'),
  }));
  console.log("Json",json);
  const style = {
    width: '260px',
  };
  const items: MenuProps['items'] = json;
  return <ProfileDropdownReporteesAtom items={items} style={style} />;
};
export default ProfileDropdownReportees;

import { MenuProps, UserCard } from 'hds-ui/components';
import { TEAM_LIST, TENANT_REVIEW_LIST } from '../../utils/queries';
import { useQueryHook } from '../../utils/client';
import exclamation from '../../assets/exclamation.svg';
import check from '../../assets/check.svg';
import { Link } from 'react-router-dom';
import './ProfileDropdownReportees.scss';
import ProfileDropdownReporteesAtom from '../../atoms/ProfileDropdownReporteesAtom';
import { APP_URL } from '../../utils/constants';

const ProfileDropDownRI = () => {
  const { data, error } = useQueryHook(TENANT_REVIEW_LIST);
  if (error) {
    return <></>;
  }
  console.log("ProfileDropDownRI",data)
  const currentURL = window.location.href;
  const TeamData = data?.tenantReviewsList?.result[3].reviews;
  console.log("TeamData>>>>>>>>>",TeamData);
  const json = TeamData?.map((data: any, index: number) => ({
    key: (index + 1).toString(),
    label: (
      <Link
        to={
          currentURL?.includes('final')
            ? `${APP_URL}/reviewer-assessment/${data?.id}`
            : `${APP_URL}/reviewer-assessment/${data?.id}`
        }>
        <div style={{ display: 'flex' }}>
          {data?.selfAssessment?.managerAssessment?.status === 'FINAL' ? (
            <img src={check} className="dropdown-status" />
          ) : (
            <img src={exclamation} className="dropdown-status" />
          )}
           <UserCard
            profilePic={data?.selfAssessment?.hierarchy?.owner?.profile?.profilePic}
            band={data?.selfAssessment?.band?.name}
            title={data?.selfAssessment?.hierarchy?.owner?.profile?.name}
            subTitle={data?.selfAssessment?.hierarchy?.owner?.profile?.designation}
            loading={false}
            imgSize="md"
            imgHoverZoom={false}
          />
        </div>
      </Link>
    ),
    disabled:
      !data?.selfAssessment?.id ||
      (currentURL?.includes('final') &&
        data?.selfAssessment?.rtDecision?.status != 'FINAL'),
  }));
  console.log("Json",json);
  const style = {
    width: '260px',
  };
  const items: MenuProps['items'] = json;
  return <ProfileDropdownReporteesAtom items={items} style={style} />;
};
export default ProfileDropDownRI;













