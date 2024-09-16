import './ProfileCardWholeTemplate.scss';
import ProfileCardWithDropdown from '../../modules/ProfileCardWithDropdown/ProfileCardWithDropdown';
import ProfilePreviousExperience from '../../molecules/ProfilePreviousExperience/ProfilePreviousExperience';
import ProfileUtilisationScore from '../../molecules/ProfileUtilisationScore/ProfileUtilisationScore';
import { getProperty } from '../../utils';
import ProfileManagerAndPreviousRTCard from '../../molecules/ProfileManagerAndPreviousRTCard/ProfileManagerAndPreviousRTCard';
import ProfileReviewersAnd360 from '../../molecules/ProfileReviewersAnd360';
import useProfileDropdownReporteesActions from '../../actions/ProfileDropdownReporteesActions';
import { useEffect } from 'react';
import { UserCard } from 'hds-ui/components';
import ProfilePreviousRTCard from '../../molecules/ProfilePreviousRTCard/ProfilePreviousRTCard';

interface ReviewerAssessmentProfileCardWholeTemplateProps {
  data: any;
  source: any;
}
const ReviewerAssessmentProfileCardWholeTemplate: React.FC<ReviewerAssessmentProfileCardWholeTemplateProps> = ({
    data,
    source
  }) => {

    console.log("DATA>>>>>>>>>>>>>Reviewer",data)
    // Normalize the data structure to handle both `data` and `reviews from manager component and reviewer assesment component`
    const normalizedData = data?.reviews?.[0] || data;  // Handles both cases
    const { selfAssessment: apiSelfAssessment } = normalizedData || {};
    const selfAssessment = apiSelfAssessment?.[0] || apiSelfAssessment || {};
    const { hierarchy, timeSpentOnCurrentBand } = selfAssessment || {};
    const owner = hierarchy?.owner || {};
    const profile = owner?.profile || {};
    const selfAssessmentId = selfAssessment?.id || normalizedData?.id;
  
    // const [isDrawerVisible, setIsDrawerVisible] = useState(false);
    const { getReviewerData} = useProfileDropdownReporteesActions();

    useEffect(() => {
        getReviewerData()
    }, [source])

  return (
    <>
      <div className="profile-card-main" data-testid="profile-template">
        <ProfileCardWithDropdown
        source="ReviewerAssessment"
          profile={profile}
          id={selfAssessmentId}
          email={owner?.email}
        />
        
        <div className='previous-RTCard'> <ProfilePreviousRTCard previousAssessment={""} /></div>

        <div className="prev-exp-utilisation">
            <ProfilePreviousExperience
              profile={profile}
              hierarchy={hierarchy}
              timeSpentOnCurrentBand={timeSpentOnCurrentBand}
            />
          </div>
        <div className="profile-card-sub-right">
        <div
      className="manager-coach-profile-container"
      data-testid="manager-coach-profile">
        <div className="manager-profile">
        <div className="font-size-14">Manager</div>
        <UserCard
          profilePic={selfAssessment?.manager?.profile?.profilePic}
          title={selfAssessment?.manager?.profile?.name}
          subTitle={selfAssessment?.manager?.profile?.track?.name}
          loading={false}
          imgSize="md"
          imgHoverZoom={false}
        />
      </div>
      </div>
        </div>
      </div>
    </>
  );
};

export default ReviewerAssessmentProfileCardWholeTemplate;


@import '../../styles/colors.scss';
.profile-card-main {
  margin: 1rem 1.5rem 1rem 1rem;
  background-color: $pastel4;
  border-radius: 15px;
  padding: 1.5rem 0 0.8rem 1.5rem;
  display: flex;
  .profile-btn-margin {
    padding: 8px 5px 8px 10px;
  }
  .prev-exp-utilisation {
    display: flex;
    flex-direction: column;
    padding-top: 2px;
  }
}

.profile-card-sub-right {
  display: flex;
  justify-content: space-around;
  width: 100%;
}

.duration {
  display: block;
  line-height: 1; /* Ensures no extra space between lines */
  margin: 0; /* Removes default margin */
}

.manager-coach-profile-container {
    .coach-title {
      margin-top: 10px;
    }
    .manager-profile {
      height: 6.958rem;
      .font-size-14 {
        font-size: 14px;
        margin-bottom: 8px;
        font-weight: 400;
      }
      .hds-user-card {
        .ant-avatar-string {
          color: $black;
        }
      }
    }
  }
  .manager-coach-profile-container .hds-user-card {
    padding: 3px 5px 0px 0;
  }


