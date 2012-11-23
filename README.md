# Problem:
 Tags are usually strings and tagged on an object with very little meaning.
 + What if I want tags to have attributes like :color, :description, :display etc. ?
 + What if I want to apply the tag with a particular context / message?
 + What if I want to see tags that are applied to parent relations and/or child relations.

# Use Case:
  A Transaction belongs_to a Donor and belongs_to a Beneficiary. Minimum mandatory values for a transaction is :amount and :date and :donor. However, the system should "tag" the Transaction as a "Warning" if the Beneficiary is not updated. It should tag the Transaction as "Incomplete" if the Donor or Beneficiary information is missing. For example, phone number of Donor is missing or the Beneficiary does not have a registered address, etc. Note: The transaction can be saved but the user who is doing the data entry should see that this Transaction has tags "Warning" and "Incomplete" attached to it. The user should be able to see what the context of these tags is: "Incomplete: Donor phone number is missing", "Warning: Beneficiary is not updated".  When the user corrects the data, these tags should be removed / updated.

 The user should be able to apply User defined tags with a context too! What is someone wants to tag a transaction as "Suspect" with a reason "possible fraud", because the Beneficiary has not filed his returns and may be presumed as fraud.

# Solution: tagtree | mongoid-tagtree gem

# config/initializers/tagtree.rb
```ruby
TagTree.tag_class_name = :tag

# app/models/transactions.rb
class Transaction
  include TagTree

  belongs_to :donor
  belongs_to :beneficiary
end

# app/models/donor.rb
class Donor
  include TagTree

  has_many :transactions
end

# app/models/beneficiary.rb
class Beneficiary
  include TagTree

  has_many :transactions
end

# To apply tags and see the tags:

d = Donor.first
d.tag(:incomplete, "Phone Number is missing") # apply the Warning tag

b = Beneficiary.first
b.tag(:incomplete, "Address not registered")  # apply the Warning tag with color.

transaction = Transaction.new(donor: d, beneficiary: b)
transaction.tags                  # => { incomplete: [ [:donor, "Phone Number is missing], [:beneficiary, "Address not registered"] ] }
transaction.tags.full_messages    # => [ "Incomplete: Donor Phone Number is missing, Beneficiary Address not registered" ]

transaction.tag(:warning, "suspect fraud")
transaction.tags                  # => { incomplete: [ [:donor, "Phone Number is missing], [:beneficiary, "Address not registered"] ], warning: [ [:transaction, "Suspect fraud"] ] }
transaction.tags.full_messages    # => [ "Incomplete: Donor Phone Number is missing, Beneficiary Address not registered", "Warning: Transaction Suspect fraud" ]

transaction.tag(:suspect)         # => apply the suspect tag with no context. (backward compatible with string tags)
```

What if we want to modify the tag attributes themselves? We can add any dynamic field to this (provided dynamic attributes are set).

```ruby
tag = Tag.where(name: :suspect).first
tag.color = "red"            # can also be specified as "0b0b0c" or something.
tag.display_in_view = true   # some dynamic attribute.
```
