﻿
<table>
<tr>
<td>

 ![Logo](https://github.com/enisn/AutoFilterer/blob/master/content/auto_filterer_icon.png?raw=true) 

</td>
<td>

 # AutoFilterer

This project aims to create filtered endpoint without writing any of query with entity framework. Just prepare your filter model and apply it into your Db Entity.

> Basicly provides QueryString To Linq conversion.

You can visit [Wiki](../../wiki) for more documents

[![Nuget](https://img.shields.io/nuget/v/AutoFilterer?logo=nuget)](https://www.nuget.org/packages/AutoFilterer/)
[![WiKi](https://img.shields.io/badge/Visit-Wiki-orange)](../../wiki)
[![CodeFactor](https://www.codefactor.io/repository/github/enisn/autofilterer/badge)](https://www.codefactor.io/repository/github/enisn/autofilterer)
[![Build status](https://ci.appveyor.com/api/projects/status/fhsry13a6k6j712w?svg=true)](https://ci.appveyor.com/project/enisn/autofilterer)

</td>
</tr>
</table>


# Getting Started

- Install `AutoFilterer` NuGet package from [here](https://www.nuget.org/packages/AutoFilterer/).

*That's it. You don't need any initialization*
***
# Usage


## Basics
- Let say you have a Model like that:

```csharp
public class Blog
{
    public string BlogId { get; set; }
    public int CategoryId { get; set; }
    public int Priority { get; set; }
    public bool IsPublished { get; set; }
    public DateTime PublishDate { get; set; }
}
```

- Let's create a filtering DTO like that:

```csharp
public class BlogFilterDto : FilterBase<Blog>
{
    public int CategoryId { get; set; }
    public int Priority { get; set; }
    public bool? IsPublished { get; set; }
}
```

- Let's create a sample Controller and get the DTO from querystring

```csharp
using AutoFilterer.Extensions; // <-- To call extension methods
//...
public class BlogsController : ControllerBase
{
    [HttpGet]
    public IActionResult Get([FromQuery]BlogFilterDto filter)
    {
        using(var db = new MyDbContext())
        {
            var blogList = db.Blogs.ApplyFilter(filter);
            return Ok(blogList);
        }
    }
}
```

- Just send following requests to check result:

  * `/Blogs?IsPublished=False`
  * `/Blogs?CategoryId=4`
  * `/Blogs?Priority=4`
  * `/Blogs?IsPublished=True&Priority=1`
  * `/Blogs?IsPublished=True&Priority=5&CategoryId=1`

***

## Using string comparisons

- Let's add a string column to model:


```csharp
public class Blog
{
    public string BlogId { get; set; }
    public int CategoryId { get; set; }
    public string Title { get; set; } // <-- We'll work on this string field
    public int Priority { get; set; }
    public bool IsPublished { get; set; }
    public DateTime PublishDate { get; set; }
}
```

- And of course add DTO too

```csharp
public class BlogFilterDto : FilterBase<Blog>
{
    public int? CategoryId { get; set; }
    public int? Priority { get; set; }
    public string Title { get; set; } // <-- Same property name with Entity's property
    public bool? IsPublished { get; set; }
}
```

- Let's add `StringFilterOptions` attribute on the string field to search as **Contains** instead of exact value:

```csharp
public class BlogFilterDto : FilterBase<Blog>
{
    public int? CategoryId { get; set; }
    public int? Priority { get; set; }
    [StringFilterOptionsAttribute(StringFilterOption.Contains)]
    public string Title { get; set; }
    public bool? IsPublished { get; set; }
}
```

- You can send following requests to check result. That's awesome!

  * `/Blogs?Title=Hello`
  * `/Blogs?Title=Hello%20World`
  * `/Blogs?Title=a`

***

## Working With Range

You may want to search a range like bwetween two DateTimes or numbers.

- The Model is same with previous sample:

```csharp
public class Blog
{
    public string BlogId { get; set; }
    public int CategoryId { get; set; }
    public string Title { get; set; }
    public int Priority { get; set; }
    public bool IsPublished { get; set; }
    public DateTime PublishDate { get; set; }
}
```

- Just use following `Range<T>` type in your DTO:
**WARNING:** Do not use nullable types as generic type parameter in `Range<T>`. Use same type with your entity. If your property is already nullable in entity, it's ok. Just use same type with model with Range

```csharp
public class BlogFilterDto : FilterBase<Blog>
{
    public int? CategoryId { get; set; }
    public Range<int> Priority { get; set; } // <-- Careful! Do not use nullable Types as Generic Type parameter
    public string Title { get; set; }
    public bool? IsPublished { get; set; }

    public Range<DateTime> PublishDate { get; set; } // <-- Use Range<DateTime> instead of below
    //public DateTime PublishDate { get; set; }
}
```

- Now, querystring is changed a little bit. Try following requests:


  * `/Blogs?Priority.Min=4`
  * `/Blogs?PublishDate.Max=01.05.2019` 
    _// Depends on CultureInfo. If you're using Request Localization, each client must send by its own datetime format__
  * `/Blogs?Priority.Min=3&Priority.Max=5`

***

## Working with Pagination
- Just inherit your Dto from `PaginationFilter` instead of FilterBase:


```csharp
public class BlogFilterDto : PaginationFilterBase<Blog> // <-- That's all!
{
    public int? CategoryId { get; set; }
    public Range<int> Priority { get; set; }
    public string Title { get; set; }
    public bool? IsPublished { get; set; }
    public Range<DateTime> PublishDate { get; set; }
}
```

- And One more thing is required to apply paginations: **Ordering By**

```csharp
public class BlogsController : ControllerBase
{
    [HttpGet]
    public IActionResult Get([FromQuery]BlogFilterDto filter)
    {
        using(var db = new MyDbContext())
        {
            // You just need to apply an ordering before calling ApplyFilter() method
            var blogList = db.Blogs.OrderBy(o => o.PublishDate).ApplyFilter(filter);
            return Ok(blogList);
        }
    }
}
```

- You can use `page` and `perPage` querystring parameters with all other parameters:
  - `/Blogs?page=2`
  - `/Blogs?perPage=10`
  - `/Blogs?perPage=10&page=2`
  - `/Blogs?perPage=2&page=3&priority.min=3&title=h`
***

# Customizations

## Customizing QueryString

You may want to use shorter parameter names in querystring. AspNetCore provides to define querystring parameter's names with `FromQuery` attribute:


```csharp
public class BlogFilterDto : FilterBase<Blog>
{
    [FromQuery(Name="category")]   // < -- This attribute may be used to customize your querystring
    public int CategoryId { get; set; }// <--  But property name must be same with entity
    public int Priority { get; set; }
    public bool? IsPublished { get; set; }
}
```

This provides to handle `category` parameter as CategoryId:

    * `/Blogs?category=4`

## Customizing Pagination Parameters
You may want to use different names instead of `page` and `perPage`

- Just go your Filter Dto and override them:

```csharp
public class BlogFilterDto : PaginationFilterBase<Blog>
{
    public int? CategoryId { get; set; }
    public Range<int> Priority { get; set; }
    public string Title { get; set; }
    public bool? IsPublished { get; set; }
    public Range<DateTime> PublishDate { get; set; }

    [FromQuery(Name = "p")] // <-- you can set querystring name
    public override int Page { get => base.Page; set => base.Page = value; }

    [FromQuery(Name = "limit")]
    public override int PerPage { get => base.PerPage; set => base.PerPage = value; }
}
```

- You can change default values also:

```csharp
    public class BlogFilterDto : PaginationFilterBase<Blog>
    {
        public BlogFilterDto()
        {
            base.PerPage = 32; // Sets default value when object is initialized.
            // Model binder will set property after constructor if request has this parameter.
        }
        public int? CategoryId { get; set; }

        public Range<int> Priority { get; set; }

        [StringFilterOptions(StringFilterOption.Contains)]
        public string Title { get; set; }

        public bool? IsPublished { get; set; }

        public Range<DateTime> PublishDate { get; set; }
    }
```


## Swagger
Of course swagger will see your parameters and use them. 👍
