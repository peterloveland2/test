{
    "Parameters": {
        "enviro": {
            "Type": "String",
            "Default": "NONE"
        },
        "AppSyncApiName": {
            "Type": "String",
            "Default": "AppSyncSimpleTransform"
        },
        "authRoleName": {
            "Type": "String"
        },
        "unauthRoleName": {
            "Type": "String"
        },
        "DynamoDBModelTableReadIOPS": {
            "Type": "Number",
            "Default": 5,
            "Description": "The number of read IOPS the table should support."
        },
        "DynamoDBModelTableWriteIOPS": {
            "Type": "Number",
            "Default": 5,
            "Description": "The number of write IOPS the table should support."
        },
        "DynamoDBBillingMode": {
            "Type": "String",
            "Default": "PAY_PER_REQUEST",
            "AllowedValues": [
                "PAY_PER_REQUEST",
                "PROVISIONED"
            ],
            "Description": "Configure @model types to create DynamoDB tables with PAY_PER_REQUEST or PROVISIONED billing modes."
        },
        "DynamoDBEnablePointInTimeRecovery": {
            "Type": "String",
            "Default": "false",
            "AllowedValues": [
                "true",
                "false"
            ],
            "Description": "Whether to enable Point in Time Recovery on the table."
        },
        "DynamoDBEnableServerSideEncryption": {
            "Type": "String",
            "Default": "true",
            "AllowedValues": [
                "true",
                "false"
            ],
            "Description": "Enable server side encryption powered by KMS."
        },
        "S3DeploymentBucket": {
            "Type": "String",
            "Description": "An S3 Bucket name where assets are deployed"
        },
        "S3DeploymentRootKey": {
            "Type": "String",
            "Description": "An S3 key relative to the S3DeploymentBucket that points to the root of the deployment directory."
        }
    },
    "Resources": {
        "GraphQLAPI": {
            "Type": "AWS::AppSync::GraphQLApi",
            "Properties": {
                "AuthenticationType": "API_KEY",
                "Name": {
                    "Fn::Join": [
                        "",
                        [
                            {
                                "Ref": "AppSyncApiName"
                            },
                            "-",
                            {
                                "Ref": "env"
                            }
                        ]
                    ]
                },
                "AdditionalAuthenticationProviders": [
                    {
                        "AuthenticationType": "AWS_IAM"
                    }
                ]
            }
        },
        "GraphQLAPITransformerSchema3CB2AE18": {
            "Type": "AWS::AppSync::GraphQLSchema",
            "Properties": {
                "ApiId": {
                    "Fn::GetAtt": [
                        "GraphQLAPI",
                        "ApiId"
                    ]
                },
                "DefinitionS3Location": {
                    "Fn::Join": [
                        "",
                        [
                            "s3://",
                            {
                                "Ref": "S3DeploymentBucket"
                            },
                            "/",
                            {
                                "Ref": "S3DeploymentRootKey"
                            },
                            "/schema.graphql"
                        ]
                    ]
                }
            }
        },
        "GraphQLAPIDefaultApiKey215A6DD7": {
            "Type": "AWS::AppSync::ApiKey",
            "Properties": {
                "ApiId": {
                    "Fn::GetAtt": [
                        "GraphQLAPI",
                        "ApiId"
                    ]
                },
                "Description": "api key description",
                "Expires": 1690394291
            }
        },
        "GraphQLAPINONEDS95A13CF0": {
            "Type": "AWS::AppSync::DataSource",
            "Properties": {
                "ApiId": {
                    "Fn::GetAtt": [
                        "GraphQLAPI",
                        "ApiId"
                    ]
                },
                "Name": "NONE_DS",
                "Type": "NONE",
                "Description": "None Data Source for Pipeline functions"
            }
        },
        "DataStore": {
            "Type": "AWS::DynamoDB::Table",
            "Properties": {
                "KeySchema": [
                    {
                        "AttributeName": "ds_pk",
                        "KeyType": "HASH"
                    },
                    {
                        "AttributeName": "ds_sk",
                        "KeyType": "RANGE"
                    }
                ],
                "AttributeDefinitions": [
                    {
                        "AttributeName": "ds_pk",
                        "AttributeType": "S"
                    },
                    {
                        "AttributeName": "ds_sk",
                        "AttributeType": "S"
                    }
                ],
                "BillingMode": "PAY_PER_REQUEST",
                "StreamSpecification": {
                    "StreamViewType": "NEW_AND_OLD_IMAGES"
                },
                "TableName": {
                    "Fn::Join": [
                        "",
                        [
                            "AmplifyDataStore-",
                            {
                                "Fn::GetAtt": [
                                    "GraphQLAPI",
                                    "ApiId"
                                ]
                            },
                            "-",
                            {
                                "Ref": "env"
                            }
                        ]
                    ]
                },
                "TimeToLiveSpecification": {
                    "AttributeName": "_ttl",
                    "Enabled": true
                }
            },
            "UpdateReplacePolicy": "Delete",
            "DeletionPolicy": "Delete"
        },
        "AmplifyDataStoreIAMRole8DE05A49": {
            "Type": "AWS::IAM::Role",
            "Properties": {
                "AssumeRolePolicyDocument": {
                    "Statement": [
                        {
                            "Action": "sts:AssumeRole",
                            "Effect": "Allow",
                            "Principal": {
                                "Service": "appsync.amazonaws.com"
                            }
                        }
                    ],
                    "Version": "2012-10-17"
                },
                "RoleName": {
                    "Fn::Join": [
                        "",
                        [
                            "AmplifyDataStoreIAMRb752cd-",
                            {
                                "Fn::GetAtt": [
                                    "GraphQLAPI",
                                    "ApiId"
                                ]
                            },
                            "-",
                            {
                                "Ref": "env"
                            }
                        ]
                    ]
                }
            }
        },
        "DynamoDBAccess71ABE5AE": {
            "Type": "AWS::IAM::Policy",
            "Properties": {
                "PolicyDocument": {
                    "Statement": [
                        {
                            "Action": [
                                "dynamodb:BatchGetItem",
                                "dynamodb:BatchWriteItem",
                                "dynamodb:PutItem",
                                "dynamodb:DeleteItem",
                                "dynamodb:GetItem",
                                "dynamodb:Scan",
                                "dynamodb:Query",
                                "dynamodb:UpdateItem"
                            ],
                            "Effect": "Allow",
                            "Resource": [
                                {
                                    "Fn::Sub": [
                                        "arn:aws:dynamodb:${AWS::Region}:${AWS::AccountId}:table/${tablename}",
                                        {
                                            "tablename": {
                                                "Fn::Join": [
                                                    "",
                                                    [
                                                        "AmplifyDataStore-",
                                                        {
                                                            "Fn::GetAtt": [
                                                                "GraphQLAPI",
                                                                "ApiId"
                                                            ]
                                                        },
                                                        "-",
                                                        {
                                                            "Ref": "env"
                                                        }
                                                    ]
                                                ]
                                            }
                                        }
                                    ]
                                },
                                {
                                    "Fn::Sub": [
                                        "arn:aws:dynamodb:${AWS::Region}:${AWS::AccountId}:table/${tablename}/*",
                                        {
                                            "tablename": {
                                                "Fn::Join": [
                                                    "",
                                                    [
                                                        "AmplifyDataStore-",
                                                        {
                                                            "Fn::GetAtt": [
                                                                "GraphQLAPI",
                                                                "ApiId"
                                                            ]
                                                        },
                                                        "-",
                                                        {
                                                            "Ref": "env"
                                                        }
                                                    ]
                                                ]
                                            }
                                        }
                                    ]
                                }
                            ]
                        }
                    ],
                    "Version": "2012-10-17"
                },
                "PolicyName": "DynamoDBAccess71ABE5AE",
                "Roles": [
                    {
                        "Ref": "AmplifyDataStoreIAMRole8DE05A49"
                    }
                ]
            }
        },
        "User": {
            "Type": "AWS::CloudFormation::Stack",
            "Properties": {
                "TemplateURL": {
                    "Fn::Join": [
                        "",
                        [
                            "https://s3.",
                            {
                                "Ref": "AWS::Region"
                            },
                            ".",
                            {
                                "Ref": "AWS::URLSuffix"
                            },
                            "/",
                            {
                                "Ref": "S3DeploymentBucket"
                            },
                            "/",
                            {
                                "Ref": "S3DeploymentRootKey"
                            },
                            "/stacks/User.json"
                        ]
                    ]
                },
                "Parameters": {
                    "DynamoDBModelTableReadIOPS": {
                        "Ref": "DynamoDBModelTableReadIOPS"
                    },
                    "DynamoDBModelTableWriteIOPS": {
                        "Ref": "DynamoDBModelTableWriteIOPS"
                    },
                    "DynamoDBBillingMode": {
                        "Ref": "DynamoDBBillingMode"
                    },
                    "DynamoDBEnablePointInTimeRecovery": {
                        "Ref": "DynamoDBEnablePointInTimeRecovery"
                    },
                    "DynamoDBEnableServerSideEncryption": {
                        "Ref": "DynamoDBEnableServerSideEncryption"
                    },
                    "referencetotransformerrootstackenv10C5A902Ref": {
                        "Ref": "env"
                    },
                    "referencetotransformerrootstackGraphQLAPI20497F53ApiId": {
                        "Fn::GetAtt": [
                            "GraphQLAPI",
                            "ApiId"
                        ]
                    },
                    "referencetotransformerrootstackGraphQLAPINONEDS2BA9D1C8Name": {
                        "Fn::GetAtt": [
                            "GraphQLAPINONEDS95A13CF0",
                            "Name"
                        ]
                    },
                    "referencetotransformerrootstackS3DeploymentBucket7592718ARef": {
                        "Ref": "S3DeploymentBucket"
                    },
                    "referencetotransformerrootstackS3DeploymentRootKeyA71EA735Ref": {
                        "Ref": "S3DeploymentRootKey"
                    },
                    "referencetotransformerrootstackauthRoleNameFB872D50Ref": {
                        "Ref": "authRoleName"
                    },
                    "referencetotransformerrootstackunauthRoleName49F3C1FERef": {
                        "Ref": "unauthRoleName"
                    }
                }
            },
            "DependsOn": [
                "GraphQLAPITransformerSchema3CB2AE18"
            ]
        },
        "Fimly": {
            "Type": "AWS::CloudFormation::Stack",
            "Properties": {
                "TemplateURL": {
                    "Fn::Join": [
                        "",
                        [
                            "https://s3.",
                            {
                                "Ref": "AWS::Region"
                            },
                            ".",
                            {
                                "Ref": "AWS::URLSuffix"
                            },
                            "/",
                            {
                                "Ref": "S3DeploymentBucket"
                            },
                            "/",
                            {
                                "Ref": "S3DeploymentRootKey"
                            },
                            "/stacks/Fimly.json"
                        ]
                    ]
                },
                "Parameters": {
                    "DynamoDBModelTableReadIOPS": {
                        "Ref": "DynamoDBModelTableReadIOPS"
                    },
                    "DynamoDBModelTableWriteIOPS": {
                        "Ref": "DynamoDBModelTableWriteIOPS"
                    },
                    "DynamoDBBillingMode": {
                        "Ref": "DynamoDBBillingMode"
                    },
                    "DynamoDBEnablePointInTimeRecovery": {
                        "Ref": "DynamoDBEnablePointInTimeRecovery"
                    },
                    "DynamoDBEnableServerSideEncryption": {
                        "Ref": "DynamoDBEnableServerSideEncryption"
                    },
                    "referencetotransformerrootstackenv10C5A902Ref": {
                        "Ref": "env"
                    },
                    "referencetotransformerrootstackGraphQLAPI20497F53ApiId": {
                        "Fn::GetAtt": [
                            "GraphQLAPI",
                            "ApiId"
                        ]
                    },
                    "referencetotransformerrootstackS3DeploymentBucket7592718ARef": {
                        "Ref": "S3DeploymentBucket"
                    },
                    "referencetotransformerrootstackS3DeploymentRootKeyA71EA735Ref": {
                        "Ref": "S3DeploymentRootKey"
                    },
                    "referencetotransformerrootstackUserNestedStackUserNestedStackResource01084C14OutputstransformerrootstackUserQuerygetUserauth0FunctionQuerygetUserauth0FunctionAppSyncFunctionAE1DE713FunctionId": {
                        "Fn::GetAtt": [
                            "User",
                            "Outputs.transformerrootstackUserQuerygetUserauth0FunctionQuerygetUserauth0FunctionAppSyncFunctionAE1DE713FunctionId"
                        ]
                    },
                    "referencetotransformerrootstackUserNestedStackUserNestedStackResource01084C14OutputstransformerrootstackUserQuerygetUserpostAuth0FunctionQuerygetUserpostAuth0FunctionAppSyncFunction083DF5E5FunctionId": {
                        "Fn::GetAtt": [
                            "User",
                            "Outputs.transformerrootstackUserQuerygetUserpostAuth0FunctionQuerygetUserpostAuth0FunctionAppSyncFunction083DF5E5FunctionId"
                        ]
                    },
                    "referencetotransformerrootstackauthRoleNameFB872D50Ref": {
                        "Ref": "authRoleName"
                    },
                    "referencetotransformerrootstackunauthRoleName49F3C1FERef": {
                        "Ref": "unauthRoleName"
                    },
                    "referencetotransformerrootstackUserNestedStackUserNestedStackResource01084C14OutputstransformerrootstackUserMutationcreateUserinit0FunctionMutationcreateUserinit0FunctionAppSyncFunction5CF77F6AFunctionId": {
                        "Fn::GetAtt": [
                            "User",
                            "Outputs.transformerrootstackUserMutationcreateUserinit0FunctionMutationcreateUserinit0FunctionAppSyncFunction5CF77F6AFunctionId"
                        ]
                    },
                    "referencetotransformerrootstackUserNestedStackUserNestedStackResource01084C14OutputstransformerrootstackUserMutationcreateUserauth0FunctionMutationcreateUserauth0FunctionAppSyncFunction0FBF687CFunctionId": {
                        "Fn::GetAtt": [
                            "User",
                            "Outputs.transformerrootstackUserMutationcreateUserauth0FunctionMutationcreateUserauth0FunctionAppSyncFunction0FBF687CFunctionId"
                        ]
                    },
                    "referencetotransformerrootstackUserNestedStackUserNestedStackResource01084C14OutputstransformerrootstackUserMutationupdateUserinit0FunctionMutationupdateUserinit0FunctionAppSyncFunction90E2C98AFunctionId": {
                        "Fn::GetAtt": [
                            "User",
                            "Outputs.transformerrootstackUserMutationupdateUserinit0FunctionMutationupdateUserinit0FunctionAppSyncFunction90E2C98AFunctionId"
                        ]
                    },
                    "referencetotransformerrootstackUserNestedStackUserNestedStackResource01084C14OutputstransformerrootstackUserSubscriptiononCreateUserauth0FunctionSubscriptiononCreateUserauth0FunctionAppSyncFunction0FC1B29BFunctionId": {
                        "Fn::GetAtt": [
                            "User",
                            "Outputs.transformerrootstackUserSubscriptiononCreateUserauth0FunctionSubscriptiononCreateUserauth0FunctionAppSyncFunction0FC1B29BFunctionId"
                        ]
                    },
                    "referencetotransformerrootstackUserNestedStackUserNestedStackResource01084C14OutputstransformerrootstackUserSubscriptionOnCreateUserDataResolverFnSubscriptionOnCreateUserDataResolverFnAppSyncFunctionCAFEE24EFunctionId": {
                        "Fn::GetAtt": [
                            "User",
                            "Outputs.transformerrootstackUserSubscriptionOnCreateUserDataResolverFnSubscriptionOnCreateUserDataResolverFnAppSyncFunctionCAFEE24EFunctionId"
                        ]
                    }
                }
            },
            "DependsOn": [
                "GraphQLAPITransformerSchema3CB2AE18"
            ]
        },
        "UserFimly": {
            "Type": "AWS::CloudFormation::Stack",
            "Properties": {
                "TemplateURL": {
                    "Fn::Join": [
                        "",
                        [
                            "https://s3.",
                            {
                                "Ref": "AWS::Region"
                            },
                            ".",
                            {
                                "Ref": "AWS::URLSuffix"
                            },
                            "/",
                            {
                                "Ref": "S3DeploymentBucket"
                            },
                            "/",
                            {
                                "Ref": "S3DeploymentRootKey"
                            },
                            "/stacks/UserFimly.json"
                        ]
                    ]
                },
                "Parameters": {
                    "DynamoDBModelTableReadIOPS": {
                        "Ref": "DynamoDBModelTableReadIOPS"
                    },
                    "DynamoDBModelTableWriteIOPS": {
                        "Ref": "DynamoDBModelTableWriteIOPS"
                    },
                    "DynamoDBBillingMode": {
                        "Ref": "DynamoDBBillingMode"
                    },
                    "DynamoDBEnablePointInTimeRecovery": {
                        "Ref": "DynamoDBEnablePointInTimeRecovery"
                    },
                    "DynamoDBEnableServerSideEncryption": {
                        "Ref": "DynamoDBEnableServerSideEncryption"
                    },
                    "referencetotransformerrootstackenv10C5A902Ref": {
                        "Ref": "env"
                    },
                    "referencetotransformerrootstackGraphQLAPI20497F53ApiId": {
                        "Fn::GetAtt": [
                            "GraphQLAPI",
                            "ApiId"
                        ]
                    },
                    "referencetotransformerrootstackS3DeploymentBucket7592718ARef": {
                        "Ref": "S3DeploymentBucket"
                    },
                    "referencetotransformerrootstackS3DeploymentRootKeyA71EA735Ref": {
                        "Ref": "S3DeploymentRootKey"
                    },
                    "referencetotransformerrootstackUserNestedStackUserNestedStackResource01084C14OutputstransformerrootstackUserQuerygetUserauth0FunctionQuerygetUserauth0FunctionAppSyncFunctionAE1DE713FunctionId": {
                        "Fn::GetAtt": [
                            "User",
                            "Outputs.transformerrootstackUserQuerygetUserauth0FunctionQuerygetUserauth0FunctionAppSyncFunctionAE1DE713FunctionId"
                        ]
                    },
                    "referencetotransformerrootstackUserNestedStackUserNestedStackResource01084C14OutputstransformerrootstackUserQuerygetUserpostAuth0FunctionQuerygetUserpostAuth0FunctionAppSyncFunction083DF5E5FunctionId": {
                        "Fn::GetAtt": [
                            "User",
                            "Outputs.transformerrootstackUserQuerygetUserpostAuth0FunctionQuerygetUserpostAuth0FunctionAppSyncFunction083DF5E5FunctionId"
                        ]
                    },
                    "referencetotransformerrootstackauthRoleNameFB872D50Ref": {
                        "Ref": "authRoleName"
                    },
                    "referencetotransformerrootstackunauthRoleName49F3C1FERef": {
                        "Ref": "unauthRoleName"
                    },
                    "referencetotransformerrootstackGraphQLAPINONEDS2BA9D1C8Name": {
                        "Fn::GetAtt": [
                            "GraphQLAPINONEDS95A13CF0",
                            "Name"
                        ]
                    },
                    "referencetotransformerrootstackUserNestedStackUserNestedStackResource01084C14OutputstransformerrootstackUserMutationcreateUserinit0FunctionMutationcreateUserinit0FunctionAppSyncFunction5CF77F6AFunctionId": {
                        "Fn::GetAtt": [
                            "User",
                            "Outputs.transformerrootstackUserMutationcreateUserinit0FunctionMutationcreateUserinit0FunctionAppSyncFunction5CF77F6AFunctionId"
                        ]
                    },
                    "referencetotransformerrootstackUserNestedStackUserNestedStackResource01084C14OutputstransformerrootstackUserMutationcreateUserauth0FunctionMutationcreateUserauth0FunctionAppSyncFunction0FBF687CFunctionId": {
                        "Fn::GetAtt": [
                            "User",
                            "Outputs.transformerrootstackUserMutationcreateUserauth0FunctionMutationcreateUserauth0FunctionAppSyncFunction0FBF687CFunctionId"
                        ]
                    },
                    "referencetotransformerrootstackUserNestedStackUserNestedStackResource01084C14OutputstransformerrootstackUserMutationupdateUserinit0FunctionMutationupdateUserinit0FunctionAppSyncFunction90E2C98AFunctionId": {
                        "Fn::GetAtt": [
                            "User",
                            "Outputs.transformerrootstackUserMutationupdateUserinit0FunctionMutationupdateUserinit0FunctionAppSyncFunction90E2C98AFunctionId"
                        ]
                    },
                    "referencetotransformerrootstackUserNestedStackUserNestedStackResource01084C14OutputstransformerrootstackUserSubscriptiononCreateUserauth0FunctionSubscriptiononCreateUserauth0FunctionAppSyncFunction0FC1B29BFunctionId": {
                        "Fn::GetAtt": [
                            "User",
                            "Outputs.transformerrootstackUserSubscriptiononCreateUserauth0FunctionSubscriptiononCreateUserauth0FunctionAppSyncFunction0FC1B29BFunctionId"
                        ]
                    },
                    "referencetotransformerrootstackUserNestedStackUserNestedStackResource01084C14OutputstransformerrootstackUserSubscriptionOnCreateUserDataResolverFnSubscriptionOnCreateUserDataResolverFnAppSyncFunctionCAFEE24EFunctionId": {
                        "Fn::GetAtt": [
                            "User",
                            "Outputs.transformerrootstackUserSubscriptionOnCreateUserDataResolverFnSubscriptionOnCreateUserDataResolverFnAppSyncFunctionCAFEE24EFunctionId"
                        ]
                    }
                }
            },
            "DependsOn": [
                "GraphQLAPITransformerSchema3CB2AE18"
            ]
        },
        "ConnectionStack": {
            "Type": "AWS::CloudFormation::Stack",
            "Properties": {
                "TemplateURL": {
                    "Fn::Join": [
                        "",
                        [
                            "https://s3.",
                            {
                                "Ref": "AWS::Region"
                            },
                            ".",
                            {
                                "Ref": "AWS::URLSuffix"
                            },
                            "/",
                            {
                                "Ref": "S3DeploymentBucket"
                            },
                            "/",
                            {
                                "Ref": "S3DeploymentRootKey"
                            },
                            "/stacks/ConnectionStack.json"
                        ]
                    ]
                },
                "Parameters": {
                    "referencetotransformerrootstackGraphQLAPI20497F53ApiId": {
                        "Fn::GetAtt": [
                            "GraphQLAPI",
                            "ApiId"
                        ]
                    },
                    "referencetotransformerrootstackGraphQLAPINONEDS2BA9D1C8Name": {
                        "Fn::GetAtt": [
                            "GraphQLAPINONEDS95A13CF0",
                            "Name"
                        ]
                    },
                    "referencetotransformerrootstackS3DeploymentBucket7592718ARef": {
                        "Ref": "S3DeploymentBucket"
                    },
                    "referencetotransformerrootstackS3DeploymentRootKeyA71EA735Ref": {
                        "Ref": "S3DeploymentRootKey"
                    },
                    "referencetotransformerrootstackUserNestedStackUserNestedStackResource01084C14OutputstransformerrootstackUserUserDataSourceA8C4C398Name": {
                        "Fn::GetAtt": [
                            "User",
                            "Outputs.transformerrootstackUserUserDataSourceA8C4C398Name"
                        ]
                    },
                    "referencetotransformerrootstackUserNestedStackUserNestedStackResource01084C14OutputstransformerrootstackUserUserTableA7A02430Ref": {
                        "Fn::GetAtt": [
                            "User",
                            "Outputs.transformerrootstackUserUserTableA7A02430Ref"
                        ]
                    },
                    "referencetotransformerrootstackauthRoleNameFB872D50Ref": {
                        "Ref": "authRoleName"
                    },
                    "referencetotransformerrootstackunauthRoleName49F3C1FERef": {
                        "Ref": "unauthRoleName"
                    },
                    "referencetotransformerrootstackFimlyNestedStackFimlyNestedStackResource5835E14AOutputstransformerrootstackFimlyFimlyDataSourceA59F3B61Name": {
                        "Fn::GetAtt": [
                            "Fimly",
                            "Outputs.transformerrootstackFimlyFimlyDataSourceA59F3B61Name"
                        ]
                    },
                    "referencetotransformerrootstackFimlyNestedStackFimlyNestedStackResource5835E14AOutputstransformerrootstackFimlyFimlyTable757BBF82Ref": {
                        "Fn::GetAtt": [
                            "Fimly",
                            "Outputs.transformerrootstackFimlyFimlyTable757BBF82Ref"
                        ]
                    },
                    "referencetotransformerrootstackUserFimlyNestedStackUserFimlyNestedStackResource0BB8DFADOutputstransformerrootstackUserFimlyUserFimlyDataSourceF0117197Name": {
                        "Fn::GetAtt": [
                            "UserFimly",
                            "Outputs.transformerrootstackUserFimlyUserFimlyDataSourceF0117197Name"
                        ]
                    },
                    "referencetotransformerrootstackUserFimlyNestedStackUserFimlyNestedStackResource0BB8DFADOutputstransformerrootstackUserFimlyUserFimlyTable1622F66CRef": {
                        "Fn::GetAtt": [
                            "UserFimly",
                            "Outputs.transformerrootstackUserFimlyUserFimlyTable1622F66CRef"
                        ]
                    }
                }
            },
            "DependsOn": [
                "Fimly",
                "GraphQLAPITransformerSchema3CB2AE18",
                "User",
                "UserFimly"
            ]
        },
        "CustomResourcesjson": {
            "Type": "AWS::CloudFormation::Stack",
            "Properties": {
                "Parameters": {
                    "AppSyncApiId": {
                        "Fn::GetAtt": [
                            "GraphQLAPI",
                            "ApiId"
                        ]
                    },
                    "AppSyncApiName": {
                        "Ref": "AppSyncApiName"
                    },
                    "env": {
                        "Ref": "env"
                    },
                    "S3DeploymentBucket": {
                        "Ref": "S3DeploymentBucket"
                    },
                    "S3DeploymentRootKey": {
                        "Ref": "S3DeploymentRootKey"
                    }
                },
                "TemplateURL": {
                    "Fn::Join": [
                        "/",
                        [
                            "https://s3.amazonaws.com",
                            {
                                "Ref": "S3DeploymentBucket"
                            },
                            {
                                "Ref": "S3DeploymentRootKey"
                            },
                            "stacks",
                            "CustomResources.json"
                        ]
                    ]
                }
            },
            "DependsOn": [
                "GraphQLAPI",
                "GraphQLAPITransformerSchema3CB2AE18",
                "User",
                "Fimly",
                "UserFimly",
                "ConnectionStack"
            ]
        }
    },
    "Outputs": {
        "GraphQLAPIKeyOutput": {
            "Description": "Your GraphQL API ID.",
            "Value": {
                "Fn::GetAtt": [
                    "GraphQLAPIDefaultApiKey215A6DD7",
                    "ApiKey"
                ]
            },
            "Export": {
                "Name": {
                    "Fn::Join": [
                        ":",
                        [
                            {
                                "Ref": "AWS::StackName"
                            },
                            "GraphQLApiKey"
                        ]
                    ]
                }
            }
        },
        "GraphQLAPIIdOutput": {
            "Description": "Your GraphQL API ID.",
            "Value": {
                "Fn::GetAtt": [
                    "GraphQLAPI",
                    "ApiId"
                ]
            },
            "Export": {
                "Name": {
                    "Fn::Join": [
                        ":",
                        [
                            {
                                "Ref": "AWS::StackName"
                            },
                            "GraphQLApiId"
                        ]
                    ]
                }
            }
        },
        "GraphQLAPIEndpointOutput": {
            "Description": "Your GraphQL API endpoint.",
            "Value": {
                "Fn::GetAtt": [
                    "GraphQLAPI",
                    "GraphQLUrl"
                ]
            },
            "Export": {
                "Name": {
                    "Fn::Join": [
                        ":",
                        [
                            {
                                "Ref": "AWS::StackName"
                            },
                            "GraphQLApiEndpoint"
                        ]
                    ]
                }
            }
        }
    }
}
