1.Signup page
```java
package com.rishabh.quizzicalminds

import android.content.Intent
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import android.widget.Toast
import androidx.core.content.ContextCompat.startActivity
import com.google.android.gms.tasks.OnCompleteListener
import com.google.firebase.auth.AuthResult
import com.google.firebase.auth.FirebaseAuth
import com.google.firebase.database.DatabaseReference
import com.google.firebase.database.FirebaseDatabase
import com.rishabh.quizzicalminds.databinding.ActivitySignUpBinding

class SignUpActivity : AppCompatActivity() {
    private lateinit var binding: ActivitySignUpBinding
    private lateinit var auth : FirebaseAuth
    private lateinit var dbRef : DatabaseReference
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivitySignUpBinding.inflate(layoutInflater)
        setContentView(binding.root)
        supportActionBar?.hide()

        auth = FirebaseAuth.getInstance()

        binding.toSigninActivity.setOnClickListener {
            startActivity(Intent(this,LoginActivity::class.java))
        }
        binding.btnForSignup.setOnClickListener {
            val name = binding.nameET.text.toString()
            val email = binding.emailET.text.toString()
            val password = binding.passwordET.text.toString()
            if(name.isEmpty() || email.isEmpty() || password.isEmpty()){
                Toast.makeText(this,"Fill up all the credentials",Toast.LENGTH_SHORT).show()
            }else{
                auth.createUserWithEmailAndPassword(email,password)
                    .addOnCompleteListener {
                        if (it.isSuccessful){
                            dbRef = FirebaseDatabase.getInstance().getReference("users")
                            val userId = dbRef.push().key!!
                            val userInsert = UserDetailModel(email)

                            dbRef.child(userId).setValue(userInsert)

                            Toast.makeText(this,"user created",Toast.LENGTH_SHORT).show()
                            startActivity(Intent(this,LoginActivity::class.java))

                        }
                        else{
                            Toast.makeText(this,"user not created",Toast.LENGTH_SHORT).show()
                        }
                    }
            }
        }

    }

    override fun onStart() {
        super.onStart()
        if(auth.currentUser != null){
            startActivity(Intent(this,QuizicalMindsStartingActivity::class.java))
        }
    }
}
```
2.Login page
```java
package com.rishabh.quizzicalminds

import android.content.Intent
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import android.widget.Toast
import com.google.firebase.auth.FirebaseAuth
import com.google.firebase.database.DatabaseReference
import com.google.firebase.database.FirebaseDatabase
import com.rishabh.quizzicalminds.databinding.ActivityLoginBinding

class LoginActivity : AppCompatActivity() {
    private lateinit var binding: ActivityLoginBinding
    companion object{
        public lateinit var auth : FirebaseAuth
    }
    private lateinit var dbRefInserting : DatabaseReference

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityLoginBinding.inflate(layoutInflater)
        setContentView(binding.root)
        supportActionBar?.hide()

        auth = FirebaseAuth.getInstance()

        binding.toSignupActivity.setOnClickListener{
            val intent = Intent(this,SignUpActivity::class.java)
            startActivity(intent)
        }
        binding.btnForLogin.setOnClickListener {
            val email = binding.emailET.text.toString()
            val password = binding.passwordET.text.toString()
            if( email.isEmpty() || password.isEmpty()){
                Toast.makeText(this,"Fill up all the credentials", Toast.LENGTH_SHORT).show()
            }else{
                  auth.signInWithEmailAndPassword(email,password)
                      .addOnCompleteListener {
                          if(it.isSuccessful){
                              startActivity(Intent(this,QuizicalMindsStartingActivity::class.java))
                          }else{
                              Toast.makeText(this,"Failed to login", Toast.LENGTH_SHORT).show()
                          }
                      }
            }



        }


    }
    override fun onStart() {
        super.onStart()
        if(auth.currentUser != null){
            val intent = Intent(this,QuizicalMindsStartingActivity::class.java)
            startActivity(intent)
        }
    }
}
```
3.Category page
```java
package com.rishabh.quizzicalminds

import android.content.Intent
import android.os.Bundle
import androidx.fragment.app.Fragment
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import androidx.recyclerview.widget.GridLayoutManager
import androidx.recyclerview.widget.RecyclerView
import com.google.firebase.database.*
import com.google.firebase.firestore.ktx.firestore
import com.google.firebase.ktx.Firebase
import com.rishabh.quizzicalminds.databinding.FragmentCategoryBinding

class CategoryFragment : Fragment() {

    private lateinit var binding: FragmentCategoryBinding
    private lateinit var dbRef : DatabaseReference
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

    }

    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        // Inflate the layout for this fragment
        binding = FragmentCategoryBinding.inflate(layoutInflater,container,false)

        val recyclerView: RecyclerView = binding.categoryRecyclerview
        recyclerView.layoutManager = GridLayoutManager(context,2)


        val cateList = ArrayList<CategoryName>()
        dbRef = FirebaseDatabase.getInstance().getReference("category")
        dbRef.addValueEventListener(object : ValueEventListener{
            override fun onDataChange(snapshot: DataSnapshot) {
                cateList.clear()
                if(snapshot.exists()){
                    for (catSanp in snapshot.children){
                        val catData = catSanp.getValue(CategoryName::class.java)
                        cateList.add(catData!!)
                    }
                    val mAdapter = Adapter(cateList)
                    binding.categoryRecyclerview.adapter = mAdapter

                    mAdapter.setOnItemClickListener(object : Adapter.onItemClickListener{
                        override fun onItemClick(position: Int) {
                            val intent = Intent(activity,LevelActivity::class.java)
                            val just : String? = cateList[position].name
                            intent.putExtra("category",just)
                            startActivity(intent)

                        }

                    })

                }
            }

            override fun onCancelled(error: DatabaseError) {
                TODO("Not yet implemented")
            }

        })

        return binding.root
    }

    companion object {

    }
}
```

4.Account page
```java
package com.rishabh.quizzicalminds

import android.content.Intent
import android.os.Bundle
import androidx.fragment.app.Fragment
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import androidx.recyclerview.widget.GridLayoutManager
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView
import com.google.firebase.auth.ktx.auth
import com.google.firebase.database.*
import com.google.firebase.ktx.Firebase
import com.rishabh.quizzicalminds.LoginActivity.Companion.auth
import com.rishabh.quizzicalminds.databinding.FragmentAccountBinding


class AccountFragment : Fragment() {
    private lateinit var binding : FragmentAccountBinding
    private lateinit var dbRef : DatabaseReference
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

    }

    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        // Inflate the layout for this fragment
        binding = FragmentAccountBinding.inflate(layoutInflater,container,false)
        val curr = auth.currentUser?.email
        binding.accountNameTV.text = curr


        binding.accountNameTV.setOnClickListener {

        }

        binding.btnForLogout.setOnClickListener {
            Firebase.auth.signOut()
            startActivity(Intent(activity,LoginActivity::class.java))

        }

//        val itemStorage = ItemStorage(requireContext())
//        val cplusplus1 = itemStorage.getItem("c++1")

        val recyclerView: RecyclerView = binding.categoryResultRecyclerview
        recyclerView.layoutManager = LinearLayoutManager(context)


        val cateList = ArrayList<CategoryName>()
        dbRef = FirebaseDatabase.getInstance().getReference("category")
        dbRef.addValueEventListener(object : ValueEventListener {
            override fun onDataChange(snapshot: DataSnapshot) {
                cateList.clear()
                if(snapshot.exists()){
                    for (catSanp in snapshot.children){
                        val catData = catSanp.getValue(CategoryName::class.java)
                        cateList.add(catData!!)
                    }
                    val mAdapter = AdapterForResultsData(cateList)
                    binding.categoryResultRecyclerview.adapter = mAdapter

                    mAdapter.setOnItemClickListener(object : AdapterForResultsData.onItemClickListener{
                        override fun onItemClick(position: Int) {
                            val intent = Intent(activity,LevelForScoreActivity::class.java)
                            val just : String? = cateList[position].name
                            intent.putExtra("category",just)
                            startActivity(intent)
                        }

                    })

                }
            }

            override fun onCancelled(error: DatabaseError) {
                TODO("Not yet implemented")
            }

        })


        return binding.root
    }

    companion object {

    }
}
```
5. contest page
```java
package com.rishabh.quizzicalminds

import android.content.Intent
import android.os.Bundle
import androidx.fragment.app.Fragment
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.Toast
import com.bumptech.glide.Glide
import com.rishabh.quizzicalminds.databinding.FragmentContestBinding

class ContestFragment : Fragment() {

    private lateinit var binding : FragmentContestBinding


    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

    }

    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        // Inflate the layout for this fragment
        binding = FragmentContestBinding.inflate(layoutInflater,container,false)
        Glide.with(this)
            .asGif()
            .load(R.drawable.menu_contest)  // Call your GIF here (url, raw, etc.)
            .into(binding.imageViewGif)
        val codeFromEditText = binding.contestCodeET.text
        binding.contestJointbtn.setOnClickListener {

            val intent = Intent(activity,ExamActivityForContest::class.java)
            intent.putExtra("CONTESTCODE",codeFromEditText)
            temp = codeFromEditText.toString()
            Toast.makeText(activity, temp,Toast.LENGTH_SHORT).show()
            startActivity(intent)

        }
        binding.contestHostbtn.setOnClickListener {
            startActivity(Intent(activity,QuestionSetterActivity::class.java))
        }
        return binding.root
    }

    companion object {
        var temp : String? = null
    }
}
```

6.Difficulty level page
```java
package com.rishabh.quizzicalminds

import android.content.Intent
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import com.bumptech.glide.Glide
import com.rishabh.quizzicalminds.databinding.ActivityLevelBinding

class LevelActivity : AppCompatActivity() {
    private lateinit var binding: ActivityLevelBinding
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityLevelBinding.inflate(layoutInflater)
        setContentView(binding.root)

        Glide.with(this)
            .asGif()
            .load(R.drawable.grades)  // Call your GIF here (url, raw, etc.)
            .into(binding.imageViewForGif)

        val just = intent.getStringExtra("category")

        binding.level1.setOnClickListener {
            val intent = Intent(this,ExamActivity::class.java)
            val newPass = just+"1"
            intent.putExtra("level",newPass)

            startActivity(intent)
            finish()
        }
        binding.level2.setOnClickListener {
            val intent = Intent(this,ExamActivity::class.java)
            val newPass = just+"2"
            intent.putExtra("level",newPass)

            startActivity(intent)
            finish()
        }
        binding.level3.setOnClickListener {
            val intent = Intent(this,ExamActivity::class.java)
            val newPass = just+"3"
            intent.putExtra("level",newPass)

            startActivity(intent)
            finish()
        }


    }

}
```

7.Quiz taking page
```java
package com.rishabh.quizzicalminds

import android.content.Intent
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import com.google.firebase.database.*
import com.rishabh.quizzicalminds.databinding.ActivityExamBinding

class ExamActivity : AppCompatActivity() {
    private lateinit var binding: ActivityExamBinding
    private lateinit var dbRef : DatabaseReference
//    private val listForExam  = ArrayList<QuestionModel>()
    private var count:Int = 0
    private var score:Int = 0
    private var catefromintent : String? = null


    companion object{
        var listForExam : ArrayList<QuestionModel> = ArrayList()
        val selectedOptions : ArrayList<String> = ArrayList()
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityExamBinding.inflate(layoutInflater)
        setContentView(binding.root)

        val questionIdentifierInFirebase : String? = intent.getStringExtra("level")
        catefromintent = intent.getStringExtra("level")


        dbRef = FirebaseDatabase.getInstance().getReference(questionIdentifierInFirebase.toString())
        dbRef.addValueEventListener(object : ValueEventListener {
            override fun onDataChange(snapshot: DataSnapshot) {

                if(snapshot.exists()){
                    for (quesSnap in snapshot.children){
                        val quesData = quesSnap.getValue(QuestionModel::class.java)
                        if (quesData != null) {
                            listForExam.add(quesData)
                        }

                    }

                    binding.questionTV.text = listForExam.get(count).question.toString()
                    binding.option1.text = listForExam.get(count).option1.toString()
                    binding.option2.text = listForExam.get(count).option2.toString()
                    binding.option3.text = listForExam.get(count).option3.toString()
                    binding.option4.text = listForExam.get(count).option4.toString()

                    binding.option1Card.setOnClickListener {
                        nextQuestion(listForExam[count].option1.toString())
                    }
                    binding.option2Card.setOnClickListener {
                        nextQuestion(listForExam[count].option2.toString())
                    }
                    binding.option3Card.setOnClickListener {
                        nextQuestion(listForExam[count].option3.toString())
                    }
                    binding.option4Card.setOnClickListener {
                        nextQuestion(listForExam[count].option4.toString())
                    }

                }
            }

            override fun onCancelled(error: DatabaseError) {
                TODO("Not yet implemented")
            }

        })

        binding.option1Card.setOnClickListener {
            nextQuestion(listForExam[count].option1.toString())
        }
        binding.option2Card.setOnClickListener {
            nextQuestion(listForExam[count].option2.toString())
        }
        binding.option3Card.setOnClickListener {
            nextQuestion(listForExam[count].option3.toString())
        }
        binding.option4Card.setOnClickListener {
            nextQuestion(listForExam[count].option4.toString())
        }


    }

    private fun nextQuestion(i : String) {
        if(count<listForExam.size){
            if (listForExam.get(count).ans.equals(i)){
                score++
            }
        }

        count++
        if (count >= listForExam.size){
            val intent = Intent(this,ScoreActivity::class.java)
            intent.putExtra("SCORE",score.toString())
            intent.putExtra("level",catefromintent)

            startActivity(intent)
            finish()
        }
        else{
            binding.questionTV.setText(listForExam.get(count).question)
            binding.option1.setText(listForExam.get(count).option1)
            binding.option2.setText(listForExam.get(count).option2)
            binding.option3.setText(listForExam.get(count).option3)
            binding.option4.setText(listForExam.get(count).option4)

        }
        selectedOptions.add(i)

    }


}
```
8.result page 
```java
package com.rishabh.quizzicalminds

import android.content.Intent
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import androidx.recyclerview.widget.GridLayoutManager
import androidx.recyclerview.widget.RecyclerView
import com.rishabh.quizzicalminds.databinding.ActivityScoreBinding
import kotlin.system.exitProcess

class ScoreActivity : AppCompatActivity() {
    private lateinit var binding: ActivityScoreBinding
    private lateinit var listForAnswer : ArrayList<QuestionModel>
    private lateinit var selected : ArrayList<String>

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityScoreBinding.inflate(layoutInflater)
        setContentView(binding.root)

        listForAnswer = ArrayList<QuestionModel>()
        listForAnswer.addAll(ExamActivity.listForExam)

        selected = ArrayList<String>()
        selected.addAll(ExamActivity.selectedOptions)

        val cate = intent.getStringExtra("level")
        val scoreTemp = "Your Score is : "+intent.getStringExtra("SCORE").toString()+" for $cate"
        binding.scoreTV.text = scoreTemp

        val recyclerView: RecyclerView = binding.scoreRV
        recyclerView.layoutManager = GridLayoutManager(this,1)

        val mAdapter = AdapterScore(listForAnswer,selected)
        binding.scoreRV.adapter = mAdapter


        val itemStorage = ItemStorage(this)
        if (cate != null) {
            itemStorage.saveItem(cate,scoreTemp)
        }




    }

    @Deprecated("Deprecated in Java")
    override fun onBackPressed() {
        exitProcess(1)

    }

}
```

9. Contest hosting
```java
package com.rishabh.quizzicalminds

import android.content.Intent
import android.os.Bundle
import androidx.fragment.app.Fragment
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.Toast
import com.bumptech.glide.Glide
import com.rishabh.quizzicalminds.databinding.FragmentContestBinding

class ContestFragment : Fragment() {

    private lateinit var binding : FragmentContestBinding


    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

    }

    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        // Inflate the layout for this fragment
        binding = FragmentContestBinding.inflate(layoutInflater,container,false)
        Glide.with(this)
            .asGif()
            .load(R.drawable.menu_contest)  // Call your GIF here (url, raw, etc.)
            .into(binding.imageViewGif)
        val codeFromEditText = binding.contestCodeET.text
        binding.contestJointbtn.setOnClickListener {

            val intent = Intent(activity,ExamActivityForContest::class.java)
            intent.putExtra("CONTESTCODE",codeFromEditText)
            temp = codeFromEditText.toString()
            Toast.makeText(activity, temp,Toast.LENGTH_SHORT).show()
            startActivity(intent)

        }
        binding.contestHostbtn.setOnClickListener {
            startActivity(Intent(activity,QuestionSetterActivity::class.java))
        }
        return binding.root
    }

    companion object {
        var temp : String? = null
    }
}
```
10.Question hosting page 
```java
package com.rishabh.quizzicalminds

import android.content.Intent
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import android.widget.Toast
import com.google.firebase.database.*
import com.rishabh.quizzicalminds.databinding.ActivityQuestionSetterBinding

class QuestionSetterActivity : AppCompatActivity() {
    private lateinit var binding: ActivityQuestionSetterBinding
    private lateinit var dbRef : DatabaseReference
    private lateinit var dbRefInserting : DatabaseReference
    private var hostnumber : Int = 0
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityQuestionSetterBinding.inflate(layoutInflater)
        setContentView(binding.root)

        // getting the host number done able to fetch the 999.
        dbRef = FirebaseDatabase.getInstance().getReference("hostnumber")
        dbRef.addValueEventListener(object : ValueEventListener {
            override fun onDataChange(snapshot: DataSnapshot) {

                if(snapshot.exists()){
                    for (catSanp in snapshot.children){
                        val catData = catSanp.getValue(HostNumber::class.java)
                        if (catData != null) {
                            hostnumber = catData.number!!
                        }
                    }


                }
            }

            override fun onCancelled(error: DatabaseError) {
                TODO("Not yet implemented")
            }

        })

        hostnumber++
        binding.insertQuestionBtn.setOnClickListener {

            dbRefInserting = FirebaseDatabase.getInstance().getReference(hostnumber.toString())
            val ques = binding.questionSetterTV.text.toString()
            val op1 = binding.questionSetterOption1ET.text.toString()
            val op2 = binding.questionSetterOption2ET.text.toString()
            val op3 = binding.questionSetterOption3ET.text.toString()
            val op4 = binding.questionSetterOption4ET.text.toString()
            val ans = binding.questionSetterAnsET.text.toString()

            val quesId = dbRefInserting.push().key!!
            val questioninsert = QuestionModel(ques,op1,op2,op3,op4,ans)

            dbRefInserting.child(quesId).setValue(questioninsert)
                .addOnCompleteListener {
                    Toast.makeText(this,"Inserted the data",Toast.LENGTH_LONG).show()
                    binding.questionSetterTV.text.clear()
                    binding.questionSetterOption1ET.text.clear()
                    binding.questionSetterOption2ET.text.clear()
                    binding.questionSetterOption3ET.text.clear()
                    binding.questionSetterOption4ET.text.clear()
                    binding.questionSetterAnsET.text.clear()
                }.addOnFailureListener { err ->
                    Toast.makeText(this,"Failed in inserting",Toast.LENGTH_LONG).show()
                }

        }

        binding.finishInsertingQuestionBtn.setOnClickListener {
            val hostId = dbRef.push().key!!
            val hostNumberInsert = HostNumber(hostnumber+1)
            val hostNumberInsertForIntent = hostNumberInsert.number?.minus(1)

            dbRef.child(hostId).setValue(hostNumberInsert)
                .addOnCompleteListener {
//                    Toast.makeText(this, "Your host number is : $hostnumber",Toast.LENGTH_LONG).show()
                   val intent = Intent(this,HostNumberShowing::class.java)
                    intent.putExtra("HOST_NUMBER",hostNumberInsertForIntent.toString())
                    startActivity(intent)
                    finish()
                }.addOnFailureListener { err ->
                    Toast.makeText(this,"Failed in inserting",Toast.LENGTH_LONG).show()
                }

        }

    }
}
```
11.Page showing unique code for the quiz
```java
package com.rishabh.quizzicalminds

import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import com.rishabh.quizzicalminds.databinding.ActivityHostNumberShowingBinding
import kotlin.system.exitProcess

class HostNumberShowing : AppCompatActivity() {
    private lateinit var binding: ActivityHostNumberShowingBinding
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityHostNumberShowingBinding.inflate(layoutInflater)
        setContentView(binding.root)

        val hostnum = intent.getStringExtra("HOST_NUMBER")
        binding.textView2.text = hostnum



    }

    override fun onBackPressed() {

        exitProcess(1)
    }
}
```