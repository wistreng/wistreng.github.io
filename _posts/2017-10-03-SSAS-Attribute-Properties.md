---
layout: post
title: SSAS ATTRIBUTE PROPERTIES
date:   2017-10-03 10:01:27 +1300
tags: SSAS OLAP PROPERTIES
picture_frame: shadow
---

*After 'Playing' SSAS for a while, I found that OLAP is also focus on the dimension processing, and all attributes have the same properties. It is really time-consuming to search information for each particular property every time, so instead of searching everywhere, this post try to put it all under one roof.*
<!--more-->

### TERMS

1. **Attribute Hierarchy**
2. **Discrete Attributes and Contiguous Attributes** 


### ADVANCED PROPERTIES

1. **AttributeHierarchyDisplayFolder** Group the Attributes into a [FolderName]
2. **AttributeHierarchyEnabled** [True] or [False] of Attribute Hierarchies

    > *Determines whether an attribute hierarchy is generated by Analysis Services for the attribute. If the attribute hierarchy is not enabled, the attribute cannot be used in a user-defined hierarchy and the attribute hierarchy cannot be referenced in Multidimensional Expressions (MDX) statements.*
3. **AttributeHierarchyOptimizedState** [FullyOptimized] and [NotOptimized], just turn this off for less frequently used attributes.
4. **AttributeHierarchyVisible** Set the visibility of the attribute [True] or [False] to a client application
5. **DefaultMember** Specify the default member(value) of the attribute
6. **DiscretizationBucketCount** grouping *contiguous values* into sets of discrete values(boundaries)
7. **DiscretizationMethod** discretization algorithm
8. **EstimatedCount** helps the server to make the aggregations
9. **IsAggregatable** Year can be aggregated into 20s, 21s, it is true that some attributes are pre-aggregation, but I wonder what on earth is non-aggregatable attribute?

### BASIC PROPERTIES

1. **Description** The description of the attribute
2. **ID** The unique identifier of the dimension, non-editable
3. **Name** The name of the attribute
4. **Type** 

    > *The value of the Type property for an attribute determines the attribute type – and specifies the type of information contained by – that attribute. Within Analysis Services 2005, attribute types help to classify an attribute based upon its business utility or functionality. Many of the available options represent types which are used by client applications to display or support an attribute.*
5. **Usage** Regular, Key, Parent

### MISC PROPERTIES

1. **AttributeHierarchyOrdered** 

    > *If the order of the members do not matter, setting this property to False for attributes where the attribute hierarchy is enabled or high cardinality attributes can significantly increase the processing performance.*
2. **GroupingBehavior** Give a **hint** to the client application whether to encourage or discourage users to group on this attribute.
3. **InstanceSelection** Give a **hint** to the client application’s UI on the recommended means of selection of a member from the attribute.
    - None - (Default) no selection used.
    - DropDown - Appropriate for situations where the number of items is small enough to display within a dropdown list.
    - List - Appropriate for situations where the number of items is too large for a dropdown list, but not large enough to require filtering.
    - Filtered List - Most useful in scenarios where the number of items is large enough to require users to filter the items to be displayed.
    - Mandatory Filter - Appropriate for situations where the number of items is so large that the display must always be filtered.
4. **MemberNamesUnique** 


### PARENT-CHILD PROPERTIES

1. **MembersWithData** The visibility [NonLeafDataHidden] or [NonLeafDataVisible] of non-leaf members that also have data. [Working with Attributes in Parent-Child Hierarchies](https://docs.microsoft.com/en-us/sql/analysis-services/multidimensional-models/parent-child-dimension-attributes).
2. **MembersWithDataCaption** Set the value *(desc) to distinguish the name of the member which has different role. 
    `e.g., Jason - Jason(As a leaf member)`
3. **NamingTemplate** This property specifies how levels in a particular parent-child hierarchy would be named. 
    `CEO -> General Manager -> Manager -> Employee`
4. **RootMemberIf** The four selection options include the following:
    - ParentIsBlankSelfOrMissing - (Default) Only members that meet one or more of the conditions described for ParentIsBlank, ParentIsSelf, or ParentIsMissing are treated as root members.
    - ParentIsBlank - Only members with a null, a zero, or an empty string in the key column or columns are treated as root members.
    - ParentIsSelf - Only members with themselves as parents are treated as root members.
    - ParentIsMissing - Only members with parents that cannot be found are treated as root members.
5. **UnaryOperatorColumn** 

### SOURCE PROPERTIES

1. **CustomRollupColumn** Specify a column which will contain the custom rollup formula(MDX expressions).
2. **CustomRollupPropertiesColumn** Used to contain the properties of a custom rollup column. [learn more about the above two properties](https://technet.microsoft.com/en-us/library/ms174474(v=sql.105).aspx).
3. **KeyColumns** Not the surrogate key for dim, this is a composite key which identifies the relationship between attributes.
    ![KeyColumns](http://www.sqlskills.com/blogs/stacia/content/binary/productattrrel_3.gif"Attribute relationship")
4. **NameColumn** A user-friendly name of the attribute member. In the parent-child scenario, use the NameColume for Id instead of the ParentId.
5. **ValueColumn** This property identifies the column that provides the value of the attribute.

