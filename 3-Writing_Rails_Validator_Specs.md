Writing an RSpec-spec for your custom Rails model validator is pretty straightforward.

* Create a mocked model with a database table.
   1. Create the migration inside the spec to run before each test.
   2. Define the model, and add the validation you want to test.
   3. Run a drop-table on the mocked database table after each test.
* Use the above model and set the validated attribute to the values you want to test.

#### Creating the mocked model
```
def mocked_model
  Class.new(ActiveRecord::Base) do
    self.table_name = 'mocked_table'

    validates :resource, some_validation: true
  end
end

def create_mocked_table
  ActiveRecord::Base.connection.create_table :mocked_table do |t|
    t.string :resource
  end
end

def drop_mocked_table
  ActiveRecord::Base.connection.drop_table :mocked_table
end
```

#### Using the mocked model
```
describe SomeValidator, type: :model do
  # Disable transactional tests for this spec only.
  # Otherwise the spec will crash when you attempt to save your mocked model to database.
  self.use_transactional_tests = false

  before(:each) { create_mocked_table }
  after(:each) { drop_mocked_table }

  it 'should be valid for valid data' do
    model = mocked_model.new(resource: 'some valid data')
    expect(model).to be_valid
  end

  it 'should be invalid for invalid data' do
    model = mocked_model.new(resource: 'some invalid data')
    expect(model).to be_invalid
  end

  # ... etc ...
end
```

#### Important note!
I have had issues with conflicting mocked models and tables when they are using the same table name and not scoped to seperate modules. I highly recommend putting the mocked model and its setup and destroy functions inside a module, and giving the database table a new name for each spec.
