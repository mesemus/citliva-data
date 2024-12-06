# Permissions for combination of User/Workflow

version: 0.0.1

# Definitions

* Repository contains records that are composed of metadata and files
* Metadata on a single record have `sensitivity level` (all the metadata on the record have the same level)
* Metadata on a record have a `restriction level` which is combined with the sensitivity level
* A `usage` level defines the technical means how the metadata can be used 
* Files on a single record have `sensitivity level` (all the files on the record have the same sensitivity level)

# Metadata

Metadata provides information about the structures that store actual data. It enables basic searching within the repository but is not designed to replace domain-specific searches within the data.

## Sensitivity Levels for Metadata

Sensitivity levels indicate how sensitive the metadata is and what requirements users must meet to access it. Higher sensitivity levels demand stricter access controls.

### Supported Sensitivity Levels:

- **`non-sensitive`**: No restrictions; anyone can access the metadata.
- **`restricted`**: Access is limited to pre-approved users.
- **`private`**: Only the owner can access the metadata.

## Restriction Levels for Metadata

Restriction levels determine who can access metadata provided that its sensitivity level permits it.

### Available Restriction Levels:

- **`public`**: Accessible by anyone, provided the sensitivity level permits it.
- **`restricted`**: Only accessible to members of the same community or users with an explicit link. The owner can view the metadata without additional access requests.
- **`sealed`**: Similar to restricted, but even the owner must request access to the metadata.
- **`private`**: Accessible only by the owner and no one else.
  - TODO: maybe also curator in case owner is incapacitated


## Usage Restrictions on Metadata

Usage restrictions define how metadata can be used.

### Usage Levels:

- **`unrestricted`**: Metadata can be accessed directly.
- **`tre`**: Metadata can only be accessed from within a trusted research environment but no other requirements are imposed.
- **`approved workflows`**: Metadata can only be accessed from within a trusted research environment using a set of approved workflows.

## Restriction/sensitivity table

| sensitivity &rarr;<br> restriction &darr; | non-sensitive | restricted | private |
| --- | --- | --- | --- |
| public | everyone | pre-approved | owner only |
| restricted | team members | pre-approved | owner only | 
| sealed | team members | pre-approved | pre-approved |
| private | owner only | owner only | owner only |

*Note:* usage restrictions are orthogonal to this table.

## User interface / UX implications on the repository

Instead of presenting a big table like this to the user, we might create a `scenarios` and present these, with the posibility of "fine-tuning" the settings in an "Advanced access" section.

Maybe those scenarios might be configurable for each community differently (does it make sense?)

# Data

Data are the actual data files. Permissions to the data are handled through the same mechanism as the access to the metadata. 

# Permissions

```python

class WorkflowPermissions(CommunityDefaultWorkflowPermissions):
    can_create = [
        PrimaryCommunityRole("submitter"),
    ]

    # who can read metadata of the record
    can_read = [
        # depositor who is at the same time a member of the community 
        # can always read the metadata of the record

        IfInState(
            "published", 
            then_ = Matrix(
                [
                    MetadataSensitivityLevel("non-sensitive"),
                    [
                        MetadataRestrictionLevel("public"),
                        [ 
                            MetadataUsageLevel("unrestricted"),
                            AnyUser()
                        ],
                        [
                            MetadataUsageLevel("tre"),
                            AnyUser() & WithinTRE()
                        ],
                        [
                            MetadataUsageLevel("workflow"),
                            AnyUser() & WithinTRE() & WithinWorkflow()
                        ]
                    ],
                    [
                        MetadataRestrictionLevel("restricted"),
                        [ 
                            MetadataUsageLevel("unrestricted"),
                            PrimaryCommunityMembers() | WithApproval() | CommunityRecordDepositor()
                        ],
                        [
                            MetadataUsageLevel("tre"),
                            (PrimaryCommunityMembers()  | WithApproval() | CommunityRecordDepositor()) & WithinTRE()
                        ],
                        [
                            MetadataUsageLevel("workflow"),
                            (PrimaryCommunityMembers()  | WithApproval() | CommunityRecordDepositor()) & WithinTRE() & WithinWorkflow()
                        ]
                    ],
                    [
                        MetadataRestrictionLevel("sealed"),
                        [ 
                            MetadataUsageLevel("unrestricted"),
                            PrimaryCommunityMembers() | WithApproval() | CommunityRecordDepositor()
                        ],
                        [
                            MetadataUsageLevel("tre"),
                            (PrimaryCommunityMembers()  | WithApproval() | CommunityRecordDepositor()) & WithinTRE()
                        ],
                        [
                            MetadataUsageLevel("workflow"),
                            (PrimaryCommunityMembers()  | WithApproval() | CommunityRecordDepositor()) & WithinTRE() & WithinWorkflow()
                        ]
                    ],
                    [
                        MetadataRestrictionLevel("private"),
                        [ 
                            MetadataUsageLevel("unrestricted"),
                            CommunityRecordDepositor()
                        ],
                        [
                            MetadataUsageLevel("tre"),
                            CommunityRecordDepositor() & WithinTRE()
                        ],
                        [
                            MetadataUsageLevel("workflow"),
                            CommunityRecordDepositor() & WithinTRE() & WithinWorkflow()
                        ]
                    ],
                ],
                [
                    MetadataSensitivityLevel("restricted"),
                    [
                        MetadataRestrictionLevel("public"),
                        [ 
                            MetadataUsageLevel("unrestricted"),
                            WithApproval() | CommunityRecordDepositor()
                        ],
                        [
                            MetadataUsageLevel("tre"),
                            (WithApproval() | CommunityRecordDepositor()) & WithinTRE()
                        ],
                        [
                            MetadataUsageLevel("workflow"),
                            (WithApproval() | CommunityRecordDepositor()) & WithinTRE() & WithinWorkflow()
                        ]
                    ],
                    [
                        MetadataRestrictionLevel("restricted"),
                        [ 
                            MetadataUsageLevel("unrestricted"),
                            WithApproval() | CommunityRecordDepositor()
                        ],
                        [
                            MetadataUsageLevel("tre"),
                            (WithApproval() | CommunityRecordDepositor()) & WithinTRE()
                        ],
                        [
                            MetadataUsageLevel("workflow"),
                            (WithApproval() | CommunityRecordDepositor()) & WithinTRE() & WithinWorkflow()
                        ]
                    ],
                    [
                        MetadataRestrictionLevel("sealed"),
                        [ 
                            MetadataUsageLevel("unrestricted"),
                            WithApproval()
                        ],
                        [
                            MetadataUsageLevel("tre"),
                            WithApproval() & WithinTRE()
                        ],
                        [
                            MetadataUsageLevel("workflow"),
                            WithApproval() & WithinTRE() & WithinWorkflow()
                        ]
                    ],
                    [
                        MetadataRestrictionLevel("private"),
                        [ 
                            MetadataUsageLevel("unrestricted"),
                            CommunityRecordDepositor()
                        ],
                        [
                            MetadataUsageLevel("tre"),
                            CommunityRecordDepositor() & WithinTRE()
                        ],
                        [
                            MetadataUsageLevel("workflow"),
                            CommunityRecordDepositor() & WithinTRE() & WithinWorkflow()
                        ]
                    ],
                ],
                [
                    MetadataSensitivityLevel("private"),
                    [
                        MetadataRestrictionLevel("public"),
                        [ 
                            MetadataUsageLevel("unrestricted"),
                            CommunityRecordDepositor()
                        ],
                        [
                            MetadataUsageLevel("tre"),
                            CommunityRecordDepositor() & WithinTRE()
                        ],
                        [
                            MetadataUsageLevel("workflow"),
                            CommunityRecordDepositor() & WithinTRE() & WithinWorkflow()
                        ]
                    ],
                    [
                        MetadataRestrictionLevel("restricted"),
                        [ 
                            MetadataUsageLevel("unrestricted"),
                            CommunityRecordDepositor()
                        ],
                        [
                            MetadataUsageLevel("tre"),
                            CommunityRecordDepositor() & WithinTRE()
                        ],
                        [
                            MetadataUsageLevel("workflow"),
                            CommunityRecordDepositor() & WithinTRE() & WithinWorkflow()
                        ]
                    ],
                    [
                        MetadataRestrictionLevel("sealed"),
                        [ 
                            MetadataUsageLevel("unrestricted"),
                            WithApproval()
                        ],
                        [
                            MetadataUsageLevel("tre"),
                            WithApproval() & WithinTRE()
                        ],
                        [
                            MetadataUsageLevel("workflow"),
                            WithApproval() & WithinTRE() & WithinWorkflow()
                        ]
                    ],
                    [
                        MetadataRestrictionLevel("private"),
                        [ 
                            MetadataUsageLevel("unrestricted"),
                            CommunityRecordDepositor()
                        ],
                        [
                            MetadataUsageLevel("tre"),
                            CommunityRecordDepositor() & WithinTRE()
                        ],
                        [
                            MetadataUsageLevel("workflow"),
                            CommunityRecordDepositor() & WithinTRE() & WithinWorkflow()
                        ]
                    ],
                ]
```