# Starting and Stoping AWS RDS instance using CloudWatch, APIgateway and Lambda

Many times when we start a project, setting up the infra and a pre-production is the first step. My curent project uses AWS RDS to host the Postgres instance that serves our production. And we set up a docker based postgres instance for the pre-prod.

Very soon we ran into issues due to verions differences. Which drove us to set up the pre-prod Postgres instance in AWS RDS for parity. Only difference was the compute configurations.

Of course creating a database in production instance was not an option due to security concerns. Never share the same instance for your production and anything else.