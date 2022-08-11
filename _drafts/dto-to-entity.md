---
layout: post
title:  "Dto to Entity Conversion"
categories: dotnet-core
---

# Dto to Entity Conversion

Some applications use DTOs (data transfer objects) to transfer data between layers.  

They might differ from the entity they represent. 

Sometimes a property in the DTO has a different name than the property in the entity. For example, the DTO might have a property called `name` but the entity might have a property called `fullName`.

Also sometimes one or more properties of the entity are not present in the DTO. For example. For example, the entity might have property called `passwordHash` and due to security reasons the DTO does not have that property exposed.

Therefore it is important to convert between the two. In this article I'm covering the DTO to entity conversion. Taking in consideration dotnet core and Entity Framework (ORM).








Link: https://stackoverflow.com/questions/27176014/how-to-add-update-child-entities-when-updating-a-parent-entity-in-ef

Because the model that gets posted to the WebApi controller is detached from any entity-framework (EF) context, the only option is to load the object graph (parent including its children) from the database and compare which children have been added, deleted or updated. (Unless you would track the changes with your own tracking mechanism during the detached state (in the browser or wherever) which in my opinion is more complex than the following.) It could look like this:

```
public void Update(UpdateParentModel model)
{
    var existingParent = _dbContext.Parents
        .Where(p => p.Id == model.Id)
        .Include(p => p.Children)
        .SingleOrDefault();

    if (existingParent != null)
    {
        // Update parent
        _dbContext.Entry(existingParent).CurrentValues.SetValues(model);

        // Delete children
        foreach (var existingChild in existingParent.Children.ToList())
        {
            if (!model.Children.Any(c => c.Id == existingChild.Id))
                _dbContext.Children.Remove(existingChild);
        }

        // Update and Insert children
        foreach (var childModel in model.Children)
        {
            var existingChild = existingParent.Children
                .Where(c => c.Id == childModel.Id && c.Id != default(int))
                .SingleOrDefault();

            if (existingChild != null)
                // Update child
                _dbContext.Entry(existingChild).CurrentValues.SetValues(childModel);
            else
            {
                // Insert child
                var newChild = new Child
                {
                    Data = childModel.Data,
                    //...
                };
                existingParent.Children.Add(newChild);
            }
        }

        _dbContext.SaveChanges();
    }
}
```

`...CurrentValues.SetValues` can take any object and maps property values to the attached entity based on the property name. If the property names in your model are different from the names in the entity you can't use this method and must assign the values one by one.
