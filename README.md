### 2. Key Points
- **Scrum Testing**:
    - **Objective**: To validate user story integration and all system behaviors according to requirements.
    - **Examples**: Simple Unit Level Forecast Calculation, Purchase Order Creation for Direct Store Delivery (DSD), Goods Receipt Creation for WMS Orders.
    - **Environment**: DevQE.
    - **Duration**: 1 sprint.
    - **Code Deployment**: Based on the development plan, deployed daily.
    - **Jira Project**: Rxl: RXRPD, RXP: RXP.

- **Early Performance**:
    - **Objective**: To validate the non-functional quality attributes of the component without involving partners.
    - **Examples**: Handling ~2.3 billion historical demand weekly for 1050 stores without impacting performance, validating other NFR categories like accessibility and usability.
    - **Environment**: PET.
    - **Duration**: 3 to 5 sprints.
    - **Code Deployment**: Initial deployment after the first development sprint cycle, followed by weekly build deployment.
    - **Jira Project**: RXPERF.

- **System Integration Testing (SIT)**:
    - **Objective**: To test feature integration across various RxR sub-domains (Intake, Clinical Services, Fulfillment) and ensure integration layer compliance, along with accessibility testing.
    - **Examples**: Search Patient, Import Patient with Allergy & health conditions, Retrieving orders for the Patient, Performing Drug Utilization Review (DUR) for the patient, Processing a claim payment.
    - **Environment**: SIT.
    - **Duration**: 5 sprints.
    - **Code Deployment**: Initial deployment after the first development sprint cycle, followed by weekly build deployment.
    - **Jira Project**: Rxl: RXQE, RXP: RXPQE.
