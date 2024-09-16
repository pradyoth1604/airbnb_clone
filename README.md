import './ProfileCardWholeTemplate.scss';
import ProfileCardWithDropdown from '../../modules/ProfileCardWithDropdown/ProfileCardWithDropdown';
import ProfilePreviousRTCard from '../../molecules/ProfilePreviousRTCard/ProfilePreviousRTCard';
import ProfileOfManagerAndCoach from '../../molecules/ProfileOfManagerAndCoach/ProfileOfManagerAndCoach';
import ProfilePreviousExperience from '../../molecules/ProfilePreviousExperience/ProfilePreviousExperience';
import ProfileTimelineButton from '../../atoms/ProfileTimelineButton';
import ProfileUtilisationScore from '../../molecules/ProfileUtilisationScore/ProfileUtilisationScore';
import RTQuestionsDrawer from '../RTQuestionsDrawer/RTQuestionsDrawer';
import { useEffect, useState } from 'react';
import { getProperty } from '../../utils';
import useProfileDropdownReporteesActions from '../../actions/ProfileDropdownReporteesActions';
import { andCheck } from 'hds-ui/utils';
import ProfileManagerAndPreviousRTCard from '../../molecules/ProfileManagerAndPreviousRTCard/ProfileManagerAndPreviousRTCard';
import ProfileReviewersAnd360 from '../../molecules/ProfileReviewersAnd360';

interface ProfileCardWholeTemplateProps {
  data: any;
  source: string;
}

const ProfileCardWholeTemplate: React.FC<ProfileCardWholeTemplateProps> = ({
  data,
  source
}) => {
  const normalizedData = data?.reviews?.[0] || data;
  const { selfAssessment: apiSelfAssessment } = normalizedData || {};
  const selfAssessment = apiSelfAssessment?.[0] || apiSelfAssessment || {};
  const { hierarchy, timeSpentOnCurrentBand } = selfAssessment || {};
  const owner = hierarchy?.owner || {};
  const profile = owner?.profile || {};
  const selfAssessmentId = selfAssessment?.id || normalizedData?.id;

  const [isDrawerVisible, setIsDrawerVisible] = useState(false);
  const { getReviewerData, getTeamList } = useProfileDropdownReporteesActions();

  const handleRTClick = () => {
    setIsDrawerVisible(true);
  };

  useEffect(() => {
    if (source === "ManagerAssessment") {
      getTeamList();
    } else if (source === "ReviewerAssessment") {
      getReviewerData();
    }
  }, [source]);

  return (
    <>
      {source === "ReviewerAssessment" ? (
        // ReviewerAssessment UI
        <div className="profile-card-main" data-testid="profile-template">
          <ProfileCardWithDropdown
            profile={profile}
            id={selfAssessment?.id}
            email={owner?.email}
          />
          <div className="profile-card-sub-right">
            <ProfileManagerAndPreviousRTCard
              manager={hierarchy?.manager}
              previousAssessment={selfAssessment?.previousAssessment}
            />
            <div className="prev-exp-utilisation">
              <ProfileReviewersAnd360
                info={{
                  reviewers: getProperty(selfAssessment, ['allReviewers'], []),
                }}
                isSubmitted={
                  getProperty(
                    selfAssessment,
                    ['managerAssessment', 'status'],
                    '',
                  ) === 'FINAL'
                }
                manager={getProperty(hierarchy, ['manager'], {})}
              />
              <ProfileUtilisationScore
                userEmail={hierarchy?.owner?.email}
                cycleId={hierarchy?.cycle?.id}
              />
            </div>
            <div className="prev-exp-utilisation">
              <ProfileReviewersAnd360
                info={{
                  reviewers360: getProperty(selfAssessment, ['reviewers360'], []),
                }}
                isSubmitted={
                  getProperty(
                    selfAssessment,
                    ['managerAssessment', 'status'],
                    '',
                  ) === 'FINAL'
                }
                manager={getProperty(hierarchy, ['manager'], {})}
              />
              <ProfilePreviousExperience
                profile={profile}
                hierarchy={hierarchy}
                timeSpentOnCurrentBand={timeSpentOnCurrentBand}
              />
            </div>
          </div>
        </div>
      ) : (
        // ManagerAssessment UI
        <div
          className={`${
            source === "ManagerAssessment"
              ? `profile-card-main-manager`
              : `profile-card-main-reviewer`
          }`}
          data-testid="profile-template"
        >
          <ProfileCardWithDropdown
            source={source}
            profile={profile}
            id={selfAssessmentId}
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
            {andCheck(
              source === "ManagerAssessment",
              <ProfileUtilisationScore
                userEmail={hierarchy?.owner?.email}
                cycleId={hierarchy?.cycle?.id}
              />
            )}
          </div>
          <ProfileOfManagerAndCoach
            manager={hierarchy?.manager}
            coach={hierarchy?.coach}
            source={source}
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
            {andCheck(
              source === "ManagerAssessment",
              <ProfileTimelineButton
                id={owner?.id}
                className={'expectations-margin'}
                title={'Manager 1 on 1 Summary'}
              />
            )}
          </div>
        </div>
      )}
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
