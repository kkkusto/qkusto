### 2. Key Points
- **End-to-End (Product Integration) Testing**:
    - **Objective**: Validate end-to-end business flows and product integration with internal and external partners.
    - **Examples**: Initiating a refill for an imported patient, integration with systems like RFT, POS, ABC, and Relay Health.
    - **Environment**: E2E.
    - **Duration**: 3 sprints.
    - **Code Deployment**: Initial deployment after the 5th development sprint and after the first E2E test cycle completion.
    - **Jira Project**: RXI, RXP, RXPQE.

- **Non-Functional Testing**:
    - **Objective**: Validate the non-functional quality attributes of the product.
    - **Examples**: Handling 5 million users without impacting performance, and validating attributes like performance, accessibility, usability, reliability, regulatory compliance, and capacity.
    - **Environment**: Perf.
    - **Duration**: 2 sprints.
    - **Code Deployment**: Initial deployment after the first E2E test cycle, followed by a patch if required.
    - **Jira Project**: RXPERF.

- **Regression Testing**:
    - **Objective**: Ensure that new capabilities, defect fixes, and configuration changes do not break previously developed product capabilities.
    - **Examples**: Search Patient, Scan Via Refill, Search Prescription.
    - **Environment**: E2E.
    - **Duration**: 1 sprint.
    - **Jira Project**: RXI, RXP, RXPQE.
