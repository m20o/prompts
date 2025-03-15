## How to implement

The introduction of new functionalities should be **incremental** and take very small verifiable steps. 

While creating code, you MUST follow the `outside-in` approach:

1. Ensure that the application works by running all the tests.

2. If some test does not work, stop and ask me what to do.

3. First, create/modify the REST API that implements the contract. You can create mock domain services.

4. Ensure that the REST API works as expected by creating tests. Stop here and ask me to move on.

5. Then, you'll implement/modify domain services one by one. 

6. For every domain service, use mocked repositories.

7. Please verify that the domain service works by creating tests and making them pass. Stop here and ask me to move on.

8. Finally, create/modify repositories with their integration tests.

   

Remember: every new functionality MUST be thoroughly tested before moving on.


## Rules

* Concrete repositories should always be tested using integration tests. 
  * Use TestContainers whenever possible. If not, stop and ask me what to do.
