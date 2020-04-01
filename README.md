# table: beneficiary_allocation
|||||||||
|---------|-------|-------|-------|-------|-------|--------|----------------------|
|**Schema Definition**||||||**mongo field**|**notes**|
|**field**|**type**|**PK**|**FK**|**IX**|**required**|**family_beneficiary_product**|**mapping notes**|
|id|uuid/autoincrement|Y||Y|Y||
|deprecated_id|STRING||||N|_id|(string)mongo id (can be deleted after migration)|
|created_at|DATE||||Y|_created|
|created_by|ID (note 1)||||N|n/a|use null or superId when migrating|
|modifed_at|DATE||||N|_modified|
|modified_by|ID(note1)||||N|n/a|use null or superId when migrating|
|precedence|enum ('primary', 'secondary')||||Y||
|allocation_pct|INT16(note2)||||Y|in beneficiaries array->amount|see note (note3)|
|deprecated_product_id|ID(note1)|||Y|Y|productId|Plan Year concerns? - or rely on history table?|
|role|enum-('ee','sp','dp','ch')||||Y||This is the role of the person who has the associated allocated beneficiaries|
|benefit_type|VARCHAR(100)|||Y|Y|n/a|if we are required to populate this during migration, an extra lookup is required to acquire the benefit type|
|beneficiary_id|ID(note1)||Y||N|n/a|


# table: beneficiary
|||||||||
|---------|-------|-------|-------|-------|-------|--------|----------------------|
|**Schema Definition**||||||**mongo field**|**notes**|
|**field**|**type**|**PK**|**FK**|**IX**|**required**|**family_beneficiary_product**|**mapping notes**|
|id|uuid/autoincrement|Y||Y|Y|n/a|
|created_at|DATE||||Y|_created|
|created_by|ID(note1)||||N|n/a|use null or superId when migrating|
|modifed_at|DATE||||N|_modified|
|modified_by|ID(note1)||||N|n/a|use null or superId when migrating|
|deprecated_family_id|ID(note1)||||Y|familyId|(string)mongo id (can be removed once new equivilant core entities are migrated) - evaluate whether this is necessary|
|deprecated_employee_user_id|ID(note1)||||Y|user->_id|(string)mongo id (can be removed once new equivilant core 'employee' entity is migrated).  This represents the ID of the employee who has enrolled in the product.|
|deprecated_beneficiary_user_id|ID(note1)||||Y|user->_id|(string)mongo id (can be removed once new equivilant core 'dependent' entity is migrated).  This represents the ID of a family member who was allocated as the beneficiary.|
|individual_name_first|VARCHAR(100)||||N(note4)|nameFirst|see note (note3)|
|individual_name_last|VARCHAR(100)||||N(note4)|nameLast|see note (note3)|
|individual_relationship|ENUM ('parent', 'ch', 'sp', 'dp', 'friend', 'relative', 'other' etc.)||||N(note4)|relationship|see note 3|
|individual_dob|DATE||||N(note4)|dateOfBirth|see note (note3)|
|indiviudal_ssn|VARCHAR(9)||||N|ssn|see note (note3)|
|trust_name|VARCHAR(100)||||N(note4)|trustName|see note (note3)|
|trustee_name_first|VARCHAR(100)||||N(note4)|trusteeFirstName|see note (note3)|
|trustee_name_last|VARCHAR(100)||||N(note4)|trusteeLastName|see note (note3)|
|trust_date_established|DATE||||N|n/a|
|trust_tax_number|VARCHAR(9)||||N|taxNumber|see note (note3)|
|estate_name|VARCHAR(100)||||N(note4)|n/a|
|estate_executor_name_last|VARCHAR(100)||||N(note4)|n/a|
|estate_executor_name_first|VARCHAR(100)||||N(note4)|n/a|
|estate_date_established|DATE||||N|n/a|
|estate_tax_number|VARCHAR(9)||||N(note4)|n/a|see note (note3)|
|organization_name|VARCHAR(100)||||N(note4)|orgName|
|organization_tax_number|VARCHAR(100)||||N(note4)|taxNumber|see note (note3)|
|charity_name|VARCHAR(100)||||N(note4)|n/a|
|charity_tax_number|VARCHAR(9)||||N(note4)|n/a|
|street_address|VARCHAR(100)||||Y|addressStreet|see note (note3)|
|suite|VARCHAR(100)||||N|addressSuite|see note (note3)|
|city|VARCHAR(100)||||Y|addressCity|see note (note3)|
|state|VARCHAR(100)||||Y|addressState|see note (note3)|
|country|VARCHAR(2)||||Y|n/a|
|postal_code|VARCHAR(100)||||Y|addressZipCode|see note (note3)|
|phone_number|VARCHAR(100)||||Y|phone|see note (note3)|
|type|ENUM ('individual', 'trustee', 'estate', 'organization', 'charity')||||Y|type|see note (note3)|

## Notes
1. entity type for ID fields TBD
2. allocation pct will be a decimal stored as a scaled INT
3. The mongo collection stores the beneficiaries as an array of objects, these feilds are properties within the object, we will be investigating how we populate these fields if the beneficiary is a dependent
4. From a SQL schema perspective these are technically not required, but for validation purposes the required fields will be dictated based on which type of beneficiary is selected
5. All core tables will have the option to maintain history.  This will be built into the functionality, and will be stored in a separate xxx_history table
6. Fields named with the prefix_mongo will be either removed if not needed or replaced with field that will represent the equivilant new core entity after is migrated


## Dependencies

|Titan Routes|||||
|---|----|---|----|----|
File|Type|Details|Comments|Dependencies|Estimated Deprecation Date|
hr.php|route|/hr/api/families/{id}/beneficiaries||maxwell-forms|
||route|/hr/api/families/{id}/beneficiary-products/search||maxwell-forms|
super.php|route|/super/api/family-beneficiaries-product|EDI Census Import|maxwell-edi|
||route-post/put|/super/api/family-beneficiaries-product/{id}||maxwell-edi|
||route-get|/super/api/family-beneficiaries-product/{id}||maxwell-edi|
||route-delete|/super/api/family-beneficiaries-product/{id}||maxwell-edi|
member.php|route|/member/api/families/{familyId}/beneficiary-products/search||"member| kardia"|
||route|/member/api/beneficiary-products/search||member, kardia|
||route|/member/api/family-beneficiaries-product||member, kardia, terraform|
||route|/member/api/family-beneficiaries-product/{id}||member, kardia|

# Reporting
1. reports-writer
2. cr-writer
3. reports-reader

# Titan Script/Crons
1. beneficiary-remove-extra-fields (script)
2. find-duplicate-beneficiaries (script)
3. orphanded-beneficiaries (script)

# Titan Events
1. beneficiaryAddressUpdated
2. beneficiaryAmountUpdated
3. beneficiaryBenTypeUpdated
4. beneficiaryCreated
5. beneficiaryDateUpdated
6. beneficiaryDeleted
7. beneficiaryDobUpdated
8. beneficiaryNameUpdated
9. beneficiaryPhoneUpdated
10. beneficiaryRelationshipUpdated
11. beneficiarySsnUpdated
12. beneficiaryTaxNumberUpdated
13. beneficiaryTypeUpdated

# Other Notes
1. 'CopyEmployer' functionality
2. Xerox implications
