# EFCore.Seeder
![alt text][logo]

A library to seed databases from CSV files, using .NET Core 3.0 and Entity Framework Core 3.0.0

A project based on [dpaquette/EntityFramework.Seeder](https://github.com/dpaquette/EntityFramework.Seeder), using EntityFramework Core instead of EF6, plus some improvements on how to handle resource files.

## How to install

`Install-Package EFCore.Seeder`

## How to use

### TL;DR;
1. Configure Entity Framework Core in your solution and get a DbContext ready.
2. Add CSV files to your solution and set Build Action on each one of them to ```Embedded Resource```
3. Make sure each one of the entities you want to insert or update implement the ```IEquatable<Class>``` interface. Also, if your entities include an Identity column, make sure to add the attribute ```[Ignore]``` to those, or make sure to set ```IDENTITY_INSERT``` **ON** on those tables where you want the tool to insert the identity columns you include in your CSV files.
4. Create a new ```CsvHelper.Configuration.Configuration``` that conforms to your CSV files. [Please refer to their website](https://joshclose.github.io/CsvHelper/getting-started/) for more information.
5. Create a ```ManifestConfiguration``` so ```EFCore.Seeder``` can find your CSV files.
6. Add this line: ```SeederConfiguration.ResetConfiguration(csvConfiguration, manifestConfiguration, assembly: typeof(<Assembly>).GetTypeInfo().Assembly);``` where ```<Assembly>``` is the name of one of the classes in the same assembly where the CSV files are included. ```csvConfiguration``` and ```manifestConfiguration``` are the previously created instances.
7. Add this line (and repeat as necessary): ```<DbContext>.<DbSet>.SeedDbSetIfEmpty(nameof(<Resource>));```. ```SeedDbSetIfEmpty``` is an extension method on ```DbSet```, and ```<Resource>``` is the name of the CSV file you want to seed into the database (without the extension ".csv")

### Configuration and setup

This library requires all resource files (CSV files) to be added as `Embedded Resource` in a runtime available assembly. Once that requirement is met, the seeder needs to be configured using:

`SeederConfiguration.ResetConfiguration(assembly: ResourceAssembly);`

In order to make it easier, you can reference a type in that assembly and let Reflection take care of retrieving the assembly instance

`SeederConfiguration.ResetConfiguration(assembly: typeof(<Assembly Class or Type>).GetTypeInfo().Assembly);`

Also, if the format for the CSV files needs to be changed, or the CsvHelper configuration needs some tweaking, this is where we need to pass in the new parameters:

`SeederConfiguration.ResetConfiguration(csvConfiguration: csvConfiguration, manifestConfiguration: manifestConfiguration, assembly: typeof(PayrollContext).GetTypeInfo().Assembly);`

Where `csvConfiguration` is an instance of `CsvHelper.Configuration.Configuration`, from the `CsvHelper` library, and `manifestConfiguration` is an instance of `ManifestConfiguration`. The last one is just a way to format the embedded resource file names to make them easy to find in the assembly. You can store the configuration in a json file and load it when setting everything up.

```
"manifestConfiguration": {
        "delimiterFieldName": "{delimiter}",
        "resourceFieldName": "{resource}",
        "extensionFieldName": "{extension}",
        "format": "{delimiter}{resource}{delimiter}{extension}",
        "delimiter": ".",
        "extension": "csv"
    }
```
As written in that example, the library would search for a resource file, in the assembly we indicate, which would be formatted as follows
```
.<DbSet Entity>.csv
```
Then the library would get the first ocurrence of a resource which name is as indicated. If we point to a ```Products``` manifest, we'd need to name our file ```Products.csv```. Doesn't matter which folder we put it in, or how deep in the hierarchy goes. As all embedded resources would be referenced as ```<Assembly><Path to Resource><Resource filename>```, it would be picked up without problem.

With regards to CsvHelper configuration, [please refer to their website](https://joshclose.github.io/CsvHelper/getting-started/).

### Usage

Once the Seeder is configured, all we need to do is call the appropiate extension method when accessing a DbSet for a particular type.

Let's say we have a DbContext with a `Products` DbSet. We could do the following:

`dbContext.Products.SeedDbSetIfEmpty(nameof(context.Products));`

This would assume that we have a `Products.csv` file in the configured assembly, with all the required information to load into the Products entity.

Also, please note that if we're going to update information as well as insert it, the `Products` entity must implement the `IEquatable<T>` interface, so we can use `product.Equal(other)` when finding the right entity to update. This is due to Entity Framework Core not having an `AddOrUpdate` method, so we need to either use the method `Add` or the method `Update`.

Please check out the test projects for more information on how to use the library.

[logo]: https://github.com/DanielAltamirano/EFCore.Seeder/raw/master/EFSeederIcon.png "EFCore.Seeder"
