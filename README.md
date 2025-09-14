# mogodb_task
Database Design of zenclass
1.users collection
{
  "_id": ObjectId(),
  "name": "John samrat",
  "email": "johnsamrat@example.com",
  "mobile": "9876543210"
}
2. codekata collection
{
  "_id": ObjectId(),
  "user_id": ObjectId("..."),
  "problems_solved": 45
}
3.attendance collection
{
  "_id": ObjectId(),
  "user_id": ObjectId("..."),
  "date": ISODate("2020-10-20"),
  "status": "absent"
}
4.topics collection
{
  "_id": ObjectId(),
  "name": "Node.js",
  "date": ISODate("2020-10-10")
}
5.tasks collection
{
  "_id": ObjectId(),
  "title": "Recipes App",
  "topic_id": ObjectId("..."),
  "due_date": ISODate("2020-10-15"),
  "user_id": ObjectId("..."),
  "submitted": false
}
6.company_drives collection
{
  "_id": ObjectId(),
  "name": "Google Recruitment",
  "date": ISODate("2020-10-20"),
  "students": [ObjectId("..."), ObjectId("...")]
}
7.mentors collection
{
  "_id": ObjectId(),
  "name": "rajavasanth",
  "mentees": [ObjectId("..."), ObjectId("..."), ...]
}
queries
1️.Find all the topics and tasks which are thought in the month of October
db.topics.find({
  date: {
    $gte: ISODate("2020-10-01"),
    $lte: ISODate("2020-10-31")
  }
})

db.tasks.find({
  due_date: {
    $gte: ISODate("2020-10-01"),
    $lte: ISODate("2020-10-31")
  }
})
2️.Find all the company drives which appeared between 15 Oct 2020 and 31 Oct 2020
db.company_drives.find({
  date: {
    $gte: ISODate("2020-10-15"),
    $lte: ISODate("2020-10-31")
  }
})
3️.Find all the company drives and students who appeared for the placement
db.company_drives.aggregate([
  {
    $lookup: {
      from: "users",
      localField: "students",
      foreignField: "_id",
      as: "student_details"
    }
  }
])
4️.Find the number of problems solved by each user in codekata
For a specific user:
db.codekata.findOne({ user_id: ObjectId("...") }, { problems_solved: 1 })
For all users:
db.codekata.aggregate([
  {
    $lookup: {
      from: "users",
      localField: "user_id",
      foreignField: "_id",
      as: "user_details"
    }
  },
  {
    $project: {
      _id: 0,
      user: { $arrayElemAt: ["$user_details.name", 0] },
      problems_solved: 1
    }
  }
])
5️.Find all mentors who have more than 15 mentees
db.mentors.find({
  $expr: {
    $gt: [{ $size: "$mentees" }, 15]
  }
})
6️.Find the number of users who are absent and task is not submitted
between 15 Oct 2020 and 31 Oct 2020
db.attendance.aggregate([
  {
    $match: {
      date: { $gte: ISODate("2020-10-15"), $lte: ISODate("2020-10-31") },
      status: "absent"
    }
  },
  {
    $lookup: {
      from: "tasks",
      localField: "user_id",
      foreignField: "user_id",
      as: "tasks"
    }
  },
  {
    $unwind: "$tasks"
  },
  {
    $match: {
      "tasks.submitted": false,
      "tasks.due_date": { $gte: ISODate("2020-10-15"), $lte: ISODate("2020-10-31") }
    }
  },
  {
    $group: {
      _id: "$user_id"
    }
  },
  {
    $count: "users_count"
  }
])
