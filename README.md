{
  title: '',
  render: (reviewer: ReviewerProps) => {
    const status = getProperty(reviewer, ['status'], '');
    const managerStatus = getProperty(reviewer, ['managerAssessmentStatus'], '');

    if (status === STATUSNAMES.N_A) {
      // When the status is N_A, we render nothing.
      return null;
    }

    if (managerStatus !== STATUSNAMES.final && status !== STATUSNAMES.N_A) {
      // If status is not N_A and managerAssessmentStatus is not final,
      // we render the edit symbol with appropriate logic.
      return (
        <span
          style={{ color: COLORS.editColor }}
          onClick={() => {
            navigate(`${APP_URL}/360/${reviewer.key}`, {
              state: {
                fromTable: true,
              },
            });
          }}
        >
          Edit
        </span>
      );
    }

    return null;
  },
  key: 'edit',
  width: '5%',
},
