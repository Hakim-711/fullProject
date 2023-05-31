here is kotlin files : 
package com.example.collegecommapp

import android.content.ContentValues
import android.content.Context
import android.content.Intent
import android.content.SharedPreferences
import android.os.Bundle
import android.util.Log
import android.widget.Toast
import androidx.appcompat.app.AppCompatActivity
import androidx.lifecycle.ViewModelProvider
import androidx.navigation.NavController
import androidx.navigation.fragment.NavHostFragment
import com.example.collegecommapp.apputils.AppUtils
import com.example.collegecommapp.auth.Login
import com.example.collegecommapp.chatclasses.ChatRoom
import com.example.collegecommapp.chatclasses.Dashboard
import com.example.collegecommapp.interfaces.Generalinterface
import com.example.collegecommapp.models.Group
import com.example.collegecommapp.sqlitedatabase.helpers.UsersSqliteDatabaseHelper
import com.example.collegecommapp.sqlitedatabase.tables.GroupTable
import com.example.collegecommapp.sqlitedatabase.tables.UserTable
import com.example.collegecommapp.viewmodels.ChattingViewModel
import com.google.firebase.FirebaseException
import com.google.firebase.auth.PhoneAuthCredential
import com.google.firebase.auth.PhoneAuthOptions
import com.google.firebase.auth.PhoneAuthProvider
import java.util.concurrent.TimeUnit


class MainActivity : AppCompatActivity(), Generalinterface {
    private lateinit var navHostFragment: NavHostFragment
    private lateinit var navController: NavController
    private val TAG = "MainActivity"
    private lateinit var mCallBack: PhoneAuthProvider.OnVerificationStateChangedCallbacks
    private lateinit var sharedPreferences: SharedPreferences
    private lateinit var chattingViewModel: ChattingViewModel
    private lateinit var appUtils: AppUtils
    private lateinit var usersSqliteDatabaseHelper: UsersSqliteDatabaseHelper


    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        navHostFragment = supportFragmentManager.findFragmentById(R.id.navFrag) as NavHostFragment
        navController = navHostFragment.navController
        usersSqliteDatabaseHelper = UsersSqliteDatabaseHelper(this)

        chattingViewModel = ViewModelProvider(this).get(ChattingViewModel::class.java)

        appUtils = AppUtils(this)
        initFrag()
    }

    private fun initFrag() {
        navController.navigate(R.id.dashboard)
    }

    //get phone details
    override fun getPhoneDetails(phone: String): String {
        var resultCode: String? = null

        mCallBack = object : PhoneAuthProvider.OnVerificationStateChangedCallbacks(){
            override fun onVerificationCompleted(p0: PhoneAuthCredential) {
                var code = p0.smsCode

                if (code != null){
                    resultCode = code
                }
            }

            override fun onVerificationFailed(p0: FirebaseException) {
                Log.i(TAG, "onVerificationFailed: $p0")

            }

            override fun onCodeSent(p0: String, p1: PhoneAuthProvider.ForceResendingToken) {
                Log.i(TAG, "onCodeSent: $p0")

            }
        }
        val db = usersSqliteDatabaseHelper.readableDatabase

        val projection = arrayOf(
            UserTable.COLUMN_NAME_FIRST_NAME,
            UserTable.COLUMN_NAME_LAST_NAME,
            UserTable.COLUMN_NAME_EMAIL
        )


        val selection = "${UserTable.COLUMN_NAME_PHONE} = ?"
        val selectionArgs = arrayOf(phone)

        val cursor = db.query(
            UserTable.TABLE_NAME,
            projection,
            selection,
            selectionArgs,
            null,
            null,
            null
        )

        val details = if (cursor.moveToFirst()) {
            val userName =
                cursor.getString(cursor.getColumnIndexOrThrow(UserTable.COLUMN_NAME_FIRST_NAME)) +
                        " " +
                        cursor.getString(cursor.getColumnIndexOrThrow(UserTable.COLUMN_NAME_LAST_NAME))
            val userEmail =
                cursor.getString(cursor.getColumnIndexOrThrow(UserTable.COLUMN_NAME_EMAIL))

            "Name: $userName, Email: $userEmail"
        } else {
            "User not found."
        }

        cursor.close()






        return resultCode!!
    }

    override fun goToProfile() {
        navController.navigate(R.id.profile)
    }

    override fun goToNewChatRooms() {
        navController.navigate(R.id.newChatRooms)
    }

    override fun goToSearch() {
        navController.navigate(R.id.search)
    }

    override fun logOut() {
        sharedPreferences = this.getSharedPreferences(getString(R.string.User), MODE_PRIVATE)
        var editor: SharedPreferences.Editor = sharedPreferences!!.edit()
        editor.clear()
        editor.apply()
        startActivity(Intent(this, Login::class.java))
        finish()
    }

    override fun addChatRoom(group: Group) {
        if (appUtils.checkWiFi()){
            var sharedPreferences2 = getSharedPreferences("USER", Context.MODE_PRIVATE)
            var userPhone = sharedPreferences2.getString(getString(R.string.phone), "")

            mCallBack = object : PhoneAuthProvider.OnVerificationStateChangedCallbacks(){
                override fun onVerificationCompleted(p0: PhoneAuthCredential) {
                    var code = p0.smsCode

                    if (code != null){
                        var newSharedPreferences: SharedPreferences = getSharedPreferences("CODE", MODE_PRIVATE)
                        var editor2: SharedPreferences.Editor = newSharedPreferences.edit()
                        editor2.putString("NUMBER", code)
                        editor2.putString("GroupId", group.group_id)
                        editor2.apply()

                        navController.navigate(R.id.verification)
                    }

                }

                override fun onVerificationFailed(p0: FirebaseException) {
                    Log.i(TAG, "onVerificationFailed: $p0")

                }

                override fun onCodeSent(p0: String, p1: PhoneAuthProvider.ForceResendingToken) {
                    Log.i(TAG, "onCodeSent: $p0")

                }
            }


        }
        else{
            Toast.makeText(this, "Connect to the internet for code generation", Toast.LENGTH_LONG).show()
        }

    }

    override fun goToMainPage() {
        navController.navigate(R.id.dashboard)
    }

    override fun goToChatPage(groupId: String) {
        var sharedPrefs: SharedPreferences = getSharedPreferences("GROUPID", MODE_PRIVATE)
        var ed: SharedPreferences.Editor = sharedPrefs.edit()
        ed.putString("groupId", groupId)
        ed.apply()

        navController.navigate(R.id.chatRoom)
    }

    override fun onBackPressed() {
        navHostFragment.childFragmentManager.primaryNavigationFragment?.let {
            if (it is ChatRoom){
                navController.navigate(R.id.dashboard)
            }
            if (it is Dashboard){
                logOut()
            }
            else{
                super.onBackPressed()
            }
        }
    }
}
/*
class MainActivity : AppCompatActivity(), Generalinterface {
private lateinit var navHostFragment: NavHostFragment
private lateinit var navController: NavController
private lateinit var sharedPreferences: SharedPreferences
private lateinit var usersSqliteDatabaseHelper: UsersSqliteDatabaseHelper
private lateinit var chattingViewModel: ChattingViewModel

override fun onCreate(savedInstanceState: Bundle?) {
super.onCreate(savedInstanceState)
setContentView(R.layout.activity_main)
navHostFragment = supportFragmentManager.findFragmentById(R.id.navFrag) as NavHostFragment
navController = navHostFragment.navController
chattingViewModel = ViewModelProvider(this).get(ChattingViewModel::class.java)
sharedPreferences = this.getSharedPreferences(getString(R.string.User), Context.MODE_PRIVATE)
usersSqliteDatabaseHelper = UsersSqliteDatabaseHelper(this)

initFrag()
}

private fun initFrag() {
navController.navigate(R.id.dashboard)
}
override fun getPhoneDetails(userPhone: String): String? {
val db = usersSqliteDatabaseHelper.readableDatabase

val projection = arrayOf(
UserTable.COLUMN_NAME_FIRST_NAME,
UserTable.COLUMN_NAME_LAST_NAME,
UserTable.COLUMN_NAME_EMAIL
)

val selection = "${UserTable.COLUMN_NAME_PHONE} = ?"
val selectionArgs = arrayOf(userPhone)

val cursor = db.query(
UserTable.TABLE_NAME,
projection,
selection,
selectionArgs,
null,
null,
null
)

val details = if (cursor.moveToFirst()) {
val userName =
cursor.getString(cursor.getColumnIndexOrThrow(UserTable.COLUMN_NAME_FIRST_NAME)) +
" " +
cursor.getString(cursor.getColumnIndexOrThrow(UserTable.COLUMN_NAME_LAST_NAME))
val userEmail =
cursor.getString(cursor.getColumnIndexOrThrow(UserTable.COLUMN_NAME_EMAIL))

"Name: $userName, Email: $userEmail"
} else {
"User not found."
}

cursor.close()

return details
}
override fun goToProfile() {

navController.navigate(R.id.profile)

}

override fun goToNewChatRooms() {
navController.navigate(R.id.newChatRooms)
}

override fun goToSearch() {
navController.navigate(R.id.search)
}


override fun logOut() {
val editor: SharedPreferences.Editor = sharedPreferences.edit()
editor.clear()
editor.apply()
startActivity(Intent(this, Login::class.java))
finish()
}

override fun addChatRoom(group: Group) {
// Get a writable database instance
val db = usersSqliteDatabaseHelper.writableDatabase

// Prepare the values to insert into the database
val values = ContentValues().apply {
put(GroupTable.COLUMN_NAME_GROUP_ID, group.group_id)
put(GroupTable.COLUMN_NAME_GROUP_NAME, group.group_name)
put(GroupTable.COLUMN_NAME_DESCRIPTION, group.group_description)
put(GroupTable.COLUMN_NAME_CAPACITY, group.group_capacity)
put(GroupTable.COLUMN_NAME_IMAGE, group.group_image)
put(GroupTable.COLUMN_NAME_CREATEDBY, group.group_created_by)
put(GroupTable.COLUMN_NAME_DATE_CREATED, group.group_date_created)
}

// Insert the new chat room into the database
val newRowId = db.insert(GroupTable.TABLE_NAME, null, values)

if (newRowId != -1L) {
// New chat room added successfully
// You can perform any additional actions here, such as updating the UI or showing a success message
} else {
// Failed to add the new chat room
// You can handle the error or show an error message
}
}


override fun goToChatPage(groupId: String) {
val sharedPrefs: SharedPreferences = getSharedPreferences("GROUPID", Context.MODE_PRIVATE)
val ed: SharedPreferences.Editor = sharedPrefs.edit()
ed.putString("groupId", groupId)
ed.apply()

navController.navigate(R.id.chatRoom)
}





override fun goToMainPage() {
navController.navigate(R.id.dashboard)
}



override fun onBackPressed() {
navHostFragment.childFragmentManager.primaryNavigationFragment?.let {
if (it is ChatRoom) {
navController.navigate(R.id.dashboard)
} else if (it is Dashboard) {
logOut()
} else {
super.onBackPressed()
}
}
}

}
*/
package com.example.collegecommapp.viewmodels

import android.app.Application
import androidx.lifecycle.AndroidViewModel
import androidx.lifecycle.viewModelScope
import com.example.collegecommapp.models.User
import com.example.collegecommapp.reporitories.AuthRepository
import com.example.collegecommapp.reporitories.MembersRepository
import kotlinx.coroutines.launch

class RegisterActivityViewModel(application: Application): AndroidViewModel(application) {
    private var authRepository: AuthRepository? = null
    private var membersRepository: MembersRepository? = null
    private var result: Long? = null
    private var user:User? = null
    private var updateCode: Int? = null
    private var lst: ArrayList<User>? = null
    private val TAG = "RegisterActivityViewMod"

    init {
        authRepository = AuthRepository.setInstance(application)
        membersRepository = MembersRepository.setInstance(application)
    }

    //register user

    fun createAccount(user: User): Long{
        viewModelScope.launch {
            result = authRepository!!.createAccount(user)
        }

        return result!!
    }

    //login user

    fun loginUser(email: String, password: String): User{
        viewModelScope.launch {
            user = authRepository!!.loginUser(email, password)
        }

        return user!!
    }

    //update user

    fun updateUser(userId: String, firstName: String, lastName: String, phone: String, password: String): Int{
        viewModelScope.launch {
            updateCode = authRepository!!.updateUser(userId, firstName, lastName, phone, password)
        }

        return updateCode!!
    }

    //retrieve users

    fun getAllUsers(): ArrayList<User>{
        viewModelScope.launch {
            lst = authRepository!!.getUsers()
        }

        return lst!!
    }


}

package com.example.collegecommapp.viewmodels

import android.app.Application
import android.util.Log
import androidx.lifecycle.AndroidViewModel
import androidx.lifecycle.MutableLiveData
import androidx.lifecycle.viewModelScope
import com.example.collegecommapp.models.Chat
import com.example.collegecommapp.models.Group
import com.example.collegecommapp.models.GroupDisplay
import com.example.collegecommapp.models.Member
import com.example.collegecommapp.reporitories.ChattingRepository
import com.example.collegecommapp.reporitories.GroupRepo
import com.example.collegecommapp.reporitories.MembersRepository
import kotlinx.coroutines.launch
import java.text.SimpleDateFormat
import java.util.*
import kotlin.collections.ArrayList

class ChattingViewModel(application: Application): AndroidViewModel(application) {
    private val TAG = "ChattingViewModel"
    private var chattingRepository: ChattingRepository? = null
    private var createResponse: Long? = null
    private var memberResponse: Long? = null
    private var chatResponse: Long? = null
    private var groups: MutableLiveData<List<Group>?> = MutableLiveData()
    private var grp: MutableLiveData<Group> = MutableLiveData()
    private var grpMembers: MutableLiveData<List<Member>> = MutableLiveData()
    private var memberGroups: MutableLiveData<List<GroupDisplay>?> = MutableLiveData()
    private var groupChats: MutableLiveData<List<Chat>> = MutableLiveData()

    private var groupRepo: GroupRepo? = null
    private var membersRepo: MembersRepository? = null

    init {
        chattingRepository = ChattingRepository.setInstance(application)
        groupRepo = GroupRepo.setInstance(application)
        membersRepo = MembersRepository.setInstance(application)
    }

    //create group
    fun createGroup(group: Group): Long{
        viewModelScope.launch {
            createResponse = groupRepo!!.createGroup(group)
        }

        return createResponse!!
    }

    //generate groups
    fun getGroups(userId: String): MutableLiveData<List<Group>?> {
        var str: ArrayList<String>? = null
        var lst: ArrayList<Group>? = null
        var res: ArrayList<Group>? = null
        viewModelScope.launch {
            str = membersRepo!!.getMemberGroups(userId)
            lst = groupRepo!!.generateGroups()
            
            if (lst!!.size > 0){
                var grpLst: ArrayList<Group> = ArrayList()
                for (i in 0 until lst!!.size){
                    for (j in 0 until str!!.size){
                        if (str!![j] != lst!![i].group_id){
                            Log.i(TAG, "getGroups: ${lst!![i].group_name}")
                            grpLst.add(lst!![i])
                        }
                        else{
                            Log.i(TAG, "getGroups Not equal: ${lst!![i].group_name}")
                        }
                    }
                }

                res = grpLst
            }
            
            groups.value = lst
        }

        return groups
    }

    //group details
    fun getGroupDetails(groupId: String): MutableLiveData<Group>{
        viewModelScope.launch {
            grp.value = groupRepo!!.getGroupDetails(groupId)
        }

        return grp
    }

    //add member
    fun addMember(member: Member): Long{
        viewModelScope.launch {
            memberResponse = membersRepo!!.addMember(member)
        }

        return memberResponse!!
    }

    //group members
    fun groupMembers(groupId: String): MutableLiveData<List<Member>>{
        viewModelScope.launch {
            grpMembers.value = membersRepo!!.getMembersByGroup(groupId)
        }

        return grpMembers
    }

    //member groups
    fun getMemberGroups(userId: String): MutableLiveData<List<GroupDisplay>?> {
        var lst: ArrayList<Group>? = null
        var str: ArrayList<String>? = null
        var res: ArrayList<Group> = ArrayList()
        var chatsFinal: ArrayList<GroupDisplay>? = null

        viewModelScope.launch {
            str = membersRepo!!.getMemberGroups(userId)
            lst = groupRepo!!.generateGroups()

            if (str!!.size > 0){
                for (j in 0 until lst!!.size){
                    for (i in 0 until str!!.size){
                        if (str!![i] == lst!![j].group_id){
                            res.add(lst!![j])
                        }
                    }
                }

                if (res.size > 0){
                    var chatLst: ArrayList<GroupDisplay> = ArrayList()
                    var tot = res.size
                    for (k in 0 until tot){
                        var chats = chattingRepository!!.getChats(res[k].group_id!!)
                        if (chats.size > 0){
                            var groupDisplay: GroupDisplay = GroupDisplay()
                            groupDisplay.group_id = res[k].group_id
                            groupDisplay.group_name = res[k].group_name
                            groupDisplay.group_image = res[k].group_image
                            groupDisplay.total = chats!!.size.toString()
                            groupDisplay.message = chats!![chats!!.size - 1].message
                            groupDisplay.date = chats!![chats!!.size - 1].date
                            groupDisplay.time = chats[chats.size - 1].time

                            chatLst.add(groupDisplay)

                        }
                        else{
                            var groupDisplay: GroupDisplay = GroupDisplay()
                            groupDisplay.group_id = res[k].group_id
                            groupDisplay.group_name = res[k].group_name
                            groupDisplay.group_image = res[k].group_image
                            groupDisplay.total = "0"
                            groupDisplay.message = "No Message"
                            groupDisplay.date = SimpleDateFormat("yyyy-MM-dd").format(Date())
                            groupDisplay.time = SimpleDateFormat("hh:mm").format(Date())

                            chatLst.add(groupDisplay)
                        }

                    }

                    chatsFinal = chatLst
                }
                memberGroups.value = chatsFinal
            }
        }

        return memberGroups
    }

    fun leaveGroup(userId: String, groupId: String): Int{
        var res: Int? = null
        viewModelScope.launch {
            res = membersRepo!!.leaveGroup(userId, groupId)
        }

        return res!!
    }

    //add chat
    fun addChat(chat: Chat): Long{
        viewModelScope.launch {
            chatResponse = chattingRepository!!.addChat(chat)
        }

        return chatResponse!!
    }

    //get group chats
    fun getGroupChats(groupId: String): MutableLiveData<List<Chat>>{
        viewModelScope.launch {
            groupChats.value = chattingRepository!!.getChats(groupId)
        }

        return groupChats
    }

}
package com.example.collegecommapp.sqlitedatabase




class Database {
    companion object{
        //database
        const val DATABASE_NAME = "chatapplication.db"
        const val DATABASE_VERSION = 10
    }
}
package com.example.collegecommapp.sqlitedatabase.tables

class UserTable {
    companion object{
        //user table
        const val TABLE_NAME = "userstesttable"
        const val COLUMN_NAME_ID = "id"
        const val COLUMN_NAME_FIRST_NAME = "firstname"
        const val COLUMN_NAME_LAST_NAME = "lastname"
        const val COLUMN_NAME_EMAIL = "email"
        const val COLUMN_NAME_PASSWORD = "password"
        const val COLUMN_NAME_PHONE = "phone"
        const val COLUMN_NAME_USERID = "userid"
    }
}
package com.example.collegecommapp.sqlitedatabase.tables
class MembersTable {
    companion object{
        const val TABLE_NAME = "memberstable"
        const val COLUMN_NAME_ID = "id"
        const val COLUMN_NAME_USERID = "userid"
        const val COLUMN_NAME_GROUPID = "groupid"
        const val COLUMN_NAME_CODE = "code"
        const val COLUMN_NAME_DATE_ADDED = "dateadded"
    }
}
package com.example.collegecommapp.sqlitedatabase.tables
class GroupTable {
    companion object{
        const val TABLE_NAME = "chatgroupstable"
        const val COLUMN_NAME_ID = "id"
        const val COLUMN_NAME_GROUP_ID = "groupid"
        const val COLUMN_NAME_GROUP_NAME = "name"
        const val COLUMN_NAME_DESCRIPTION = "description"
        const val COLUMN_NAME_CAPACITY = "capacity"
        const val COLUMN_NAME_IMAGE = "image"
        const val COLUMN_NAME_CREATEDBY = "createdby"
        const val COLUMN_NAME_DATE_CREATED = "datecreated"
    }
}
package com.example.collegecommapp.sqlitedatabase.tables

class ChatsTable {
    companion object{
        const val TABLE_NAME = "chatstable"
        const val COLUMN_NAME_ID = "id"
        const val COLUMN_NAME_CHAT_ID = "chatid"
        const val COLUMN_NAME_USERID = "userid"
        const val COLUMN_NAME_MESSAGE = "message"
        const val COLUMN_NAME_TIME = "time"
        const val COLUMN_NAME_DATE = "date"
        const val COLUMN_NAME_GROUPID = "groupid"
        const val COLUMN_NAME_IMAGE = "image"
    }
}
package com.example.collegecommapp.sqlitedatabase.queries

import com.example.collegecommapp.sqlitedatabase.tables.ChatsTable

class ChatEntries {
    companion object{
        var CHAT_SQL_ENTRIES = "CREATE TABLE ${ChatsTable.TABLE_NAME} (" +
                "${ChatsTable.COLUMN_NAME_ID} INTEGER," +
                "${ChatsTable.COLUMN_NAME_CHAT_ID} TEXT," +
                "${ChatsTable.COLUMN_NAME_DATE} TEXT," +
                "${ChatsTable.COLUMN_NAME_GROUPID} TEXT," +
                "${ChatsTable.COLUMN_NAME_MESSAGE} TEXT," +
                "${ChatsTable.COLUMN_NAME_TIME} TEXT," +
                "${ChatsTable.COLUMN_NAME_USERID} TEXT," +
                "${ChatsTable.COLUMN_NAME_IMAGE} TEXT)"

        var CHAT_SQL_DROP = "DROP TABLE IF EXISTS ${ChatsTable.TABLE_NAME}"
    }
}
package com.example.collegecommapp.sqlitedatabase.queries

import com.example.collegecommapp.sqlitedatabase.tables.GroupTable

class GroupQueries {
    companion object{
        var GROUP_SQL_ENTRIES = "CREATE TABLE ${GroupTable.TABLE_NAME} (" +
                "${GroupTable.COLUMN_NAME_ID} INTEGER PRIMARY KEY AUTOINCREMENT UNIQUE," +
                "${GroupTable.COLUMN_NAME_GROUP_ID} TEXT," +
                "${GroupTable.COLUMN_NAME_GROUP_NAME} TEXT," +
                "${GroupTable.COLUMN_NAME_DESCRIPTION} TEXT," +
                "${GroupTable.COLUMN_NAME_CAPACITY} TEXT," +
                "${GroupTable.COLUMN_NAME_IMAGE} TEXT," +
                "${GroupTable.COLUMN_NAME_DATE_CREATED} TEXT," +
                "${GroupTable.COLUMN_NAME_CREATEDBY} TEXT)"

        var GROUP_DELETE_SQL = "DROP TABLE IF EXISTS ${GroupTable.TABLE_NAME}"
    }
}

package com.example.collegecommapp.sqlitedatabase.queries

import com.example.collegecommapp.sqlitedatabase.tables.MembersTable

class MembersQueries {
    companion object{
        var MEMBERS_SQL_ENTRIES = "CREATE TABLE ${MembersTable.TABLE_NAME} (" +
                "${MembersTable.COLUMN_NAME_ID} INTEGER PRIMARY KEY AUTOINCREMENT UNIQUE," +
                "${MembersTable.COLUMN_NAME_GROUPID} TEXT," +
                "${MembersTable.COLUMN_NAME_CODE} TEXT," +
                "${MembersTable.COLUMN_NAME_DATE_ADDED} TEXT," +
                "${MembersTable.COLUMN_NAME_USERID} TEXT)"


        var MEMBERS_DELETE_ENTRIES = "DROP TABLE IF EXISTS ${MembersTable.TABLE_NAME}"
    }
}
package com.example.collegecommapp.sqlitedatabase.queries

import com.example.collegecommapp.sqlitedatabase.tables.UserTable

class TableQueries {
    companion object{
        //USERS
        //create table
        var SQL_CREATE_ENTRIES = "CREATE TABLE ${UserTable.TABLE_NAME} (" +
                "${UserTable.COLUMN_NAME_ID} INTEGER PRIMARY KEY AUTOINCREMENT UNIQUE," +
                "${UserTable.COLUMN_NAME_FIRST_NAME} TEXT," +
                "${UserTable.COLUMN_NAME_LAST_NAME} TEXT," +
                "${UserTable.COLUMN_NAME_EMAIL} TEXT," +
                "${UserTable.COLUMN_NAME_PHONE} TEXT," +
                "${UserTable.COLUMN_NAME_USERID} TEXT," +
                "${UserTable.COLUMN_NAME_PASSWORD} TEXT)"

        //drop table
        var SQL_DELETE_ENTRIES = "DROP TABLE IF EXISTS ${UserTable.TABLE_NAME}"
    }
}
package com.example.collegecommapp.sqlitedatabase.helpers



import android.content.Context
import android.database.sqlite.SQLiteDatabase
import android.database.sqlite.SQLiteOpenHelper
import com.example.collegecommapp.sqlitedatabase.Database
import com.example.collegecommapp.sqlitedatabase.queries.ChatEntries
import com.example.collegecommapp.sqlitedatabase.queries.GroupQueries
import com.example.collegecommapp.sqlitedatabase.queries.MembersQueries
import com.example.collegecommapp.sqlitedatabase.queries.TableQueries


class UsersSqliteDatabaseHelper(context: Context): SQLiteOpenHelper(context, Database.DATABASE_NAME, null, Database.DATABASE_VERSION) {
    override fun onCreate(p0: SQLiteDatabase?) {
        p0!!.execSQL(TableQueries.SQL_CREATE_ENTRIES)
        p0.execSQL(GroupQueries.GROUP_SQL_ENTRIES)
        p0.execSQL(MembersQueries.MEMBERS_SQL_ENTRIES)
        p0.execSQL(ChatEntries.CHAT_SQL_ENTRIES)
    }

    override fun onUpgrade(p0: SQLiteDatabase?, p1: Int, p2: Int) {
        p0!!.execSQL(TableQueries.SQL_DELETE_ENTRIES)
        p0.execSQL(GroupQueries.GROUP_DELETE_SQL)
        p0.execSQL(MembersQueries.MEMBERS_DELETE_ENTRIES)
        p0.execSQL(ChatEntries.CHAT_SQL_DROP)
        onCreate(p0)
    }

    override fun onDowngrade(db: SQLiteDatabase?, oldVersion: Int, newVersion: Int) {
        onUpgrade(db, oldVersion, newVersion)
    }

}

package com.example.collegecommapp.reporitories

import android.annotation.SuppressLint
import android.app.Activity
import android.app.Application
import android.content.ContentValues
import android.content.Context
import android.database.sqlite.SQLiteDatabase
import android.util.Log
import com.example.collegecommapp.models.User
import com.example.collegecommapp.sqlitedatabase.helpers.UsersSqliteDatabaseHelper
import com.example.collegecommapp.sqlitedatabase.tables.UserTable


class AuthRepository(context: Context) {
    private var usersSqliteDatabaseHelper: UsersSqliteDatabaseHelper? = null

    init {
        usersSqliteDatabaseHelper = UsersSqliteDatabaseHelper(context)
    }

    companion object{
        private var INSTANCE: AuthRepository? = null

        fun setInstance(context: Context): AuthRepository{
            if (INSTANCE == null){
                INSTANCE = AuthRepository(context)
            }

            return INSTANCE!!
        }
    }

    //create account
    suspend fun createAccount(user: User): Long {
        var result: Long? = null

        val db: SQLiteDatabase = usersSqliteDatabaseHelper!!.writableDatabase
        val contentValues: ContentValues = ContentValues().apply {
            put(UserTable.COLUMN_NAME_EMAIL, user.email)
            put(UserTable.COLUMN_NAME_FIRST_NAME, user.firstName)
            put(UserTable.COLUMN_NAME_LAST_NAME, user.lastName)
            put(UserTable.COLUMN_NAME_PHONE, user.phone)
            put(UserTable.COLUMN_NAME_PASSWORD, user.password)
            put(UserTable.COLUMN_NAME_USERID, user.userId)
        }

        val rowId = db.insert(UserTable.TABLE_NAME, null, contentValues)
        if (rowId >= 0){
            result = rowId
        }
        else{
            result = -1
        }

        return result
    }


    suspend fun loginUser(email: String, password: String): User{
        val db: SQLiteDatabase = usersSqliteDatabaseHelper!!.readableDatabase
        val projection = arrayOf(UserTable.COLUMN_NAME_USERID, UserTable.COLUMN_NAME_FIRST_NAME, UserTable.COLUMN_NAME_LAST_NAME, UserTable.COLUMN_NAME_ID, UserTable.COLUMN_NAME_PHONE, UserTable.COLUMN_NAME_EMAIL, UserTable.COLUMN_NAME_PASSWORD)
        var selection = "${UserTable.COLUMN_NAME_EMAIL} = ? AND ${UserTable.COLUMN_NAME_PASSWORD} = ?"
        var selectionArgs = arrayOf(email, password)

        val cursor = db.query(UserTable.TABLE_NAME,
        projection,
        selection,
        selectionArgs,
        null,
        null,
        null)

        var user: User = User()
        while (cursor.moveToNext()){
            user.userId = cursor.getString(cursor.getColumnIndexOrThrow(UserTable.COLUMN_NAME_USERID))
            user.firstName = cursor.getString(cursor.getColumnIndexOrThrow(UserTable.COLUMN_NAME_FIRST_NAME))
            user.lastName = cursor.getString(cursor.getColumnIndexOrThrow(UserTable.COLUMN_NAME_LAST_NAME))
            user.email = cursor.getString(cursor.getColumnIndexOrThrow(UserTable.COLUMN_NAME_EMAIL))
            user.password = cursor.getString(cursor.getColumnIndexOrThrow(UserTable.COLUMN_NAME_PASSWORD))
            user.phone = cursor.getString(cursor.getColumnIndexOrThrow(UserTable.COLUMN_NAME_PHONE))
        }

        return user
    }

    suspend fun updateUser(userId: String, firstName: String, lastName: String, phone: String, password: String): Int{
        val db = usersSqliteDatabaseHelper!!.writableDatabase
        var contentValues = ContentValues().apply {
            put(UserTable.COLUMN_NAME_FIRST_NAME, firstName)
            put(UserTable.COLUMN_NAME_LAST_NAME, lastName)
            put(UserTable.COLUMN_NAME_PHONE, phone)
            put(UserTable.COLUMN_NAME_PASSWORD, password)
        }
        var selection = "${UserTable.COLUMN_NAME_USERID} LIKE ?"
        var selectionArgs = arrayOf(userId)
        val count = db.update(
            UserTable.TABLE_NAME,
            contentValues,
            selection,
            selectionArgs
        )

        if (count > 0){
            return count
        }
        else{
            return 0
        }
    }

    suspend fun getUsers(): ArrayList<User>{
        val db: SQLiteDatabase = usersSqliteDatabaseHelper!!.readableDatabase
        val projection = arrayOf(UserTable.COLUMN_NAME_USERID, UserTable.COLUMN_NAME_FIRST_NAME, UserTable.COLUMN_NAME_LAST_NAME, UserTable.COLUMN_NAME_ID, UserTable.COLUMN_NAME_PHONE, UserTable.COLUMN_NAME_EMAIL, UserTable.COLUMN_NAME_PASSWORD)

        val cursor = db.query(UserTable.TABLE_NAME,
            projection,
            null,
            null,
            null,
            null,
            null)

        val us: ArrayList<User> = ArrayList()

        while (cursor.moveToNext()){
            var user: User = User()
            user.userId = cursor.getString(cursor.getColumnIndexOrThrow(UserTable.COLUMN_NAME_USERID))
            user.firstName = cursor.getString(cursor.getColumnIndexOrThrow(UserTable.COLUMN_NAME_FIRST_NAME))
            user.lastName = cursor.getString(cursor.getColumnIndexOrThrow(UserTable.COLUMN_NAME_LAST_NAME))
            user.email = cursor.getString(cursor.getColumnIndexOrThrow(UserTable.COLUMN_NAME_EMAIL))
            user.password = cursor.getString(cursor.getColumnIndexOrThrow(UserTable.COLUMN_NAME_PASSWORD))
            user.phone = cursor.getString(cursor.getColumnIndexOrThrow(UserTable.COLUMN_NAME_PHONE))

            us.add(user)
        }

        return us
    }
}
package com.example.collegecommapp.reporitories

import android.content.ContentValues
import android.content.Context
import android.database.sqlite.SQLiteDatabase
import androidx.lifecycle.LiveData
import androidx.lifecycle.MutableLiveData
import com.example.collegecommapp.models.Chat
import com.example.collegecommapp.sqlitedatabase.helpers.UsersSqliteDatabaseHelper
import com.example.collegecommapp.sqlitedatabase.tables.ChatsTable


class ChattingRepository(context: Context) {
    private val TAG = "Dashboard"
    private var usersSqliteDatabaseHelper: UsersSqliteDatabaseHelper? = null

    init {
        usersSqliteDatabaseHelper = UsersSqliteDatabaseHelper(context)
    }

    companion object{
        private var INSTANCE: ChattingRepository? = null

        fun setInstance(context: Context): ChattingRepository{
            if (INSTANCE == null){
                INSTANCE = ChattingRepository(context)
            }

            return INSTANCE!!
        }
    }

    //CHATTING
    suspend fun addChat(chat: Chat): Long{
        var db: SQLiteDatabase = usersSqliteDatabaseHelper!!.writableDatabase
        var contentValues: ContentValues = ContentValues().apply {
            put(ChatsTable.COLUMN_NAME_CHAT_ID, chat.chatId)
            put(ChatsTable.COLUMN_NAME_USERID, chat.userId)
            put(ChatsTable.COLUMN_NAME_GROUPID, chat.groupId)
            put(ChatsTable.COLUMN_NAME_MESSAGE, chat.message)
            put(ChatsTable.COLUMN_NAME_DATE, chat.date)
            put(ChatsTable.COLUMN_NAME_TIME, chat.time)
            put(ChatsTable.COLUMN_NAME_IMAGE, chat.img)
        }

        var response = db.insert(ChatsTable.TABLE_NAME, null, contentValues)
        if (response >= 0){
            return response
        }
        else{
            return -1
        }
    }

    suspend fun getChats(groupId: String): ArrayList<Chat>{
        var db = usersSqliteDatabaseHelper!!.readableDatabase
        var projection = arrayOf(ChatsTable.COLUMN_NAME_CHAT_ID, ChatsTable.COLUMN_NAME_USERID, ChatsTable.COLUMN_NAME_TIME,
        ChatsTable.COLUMN_NAME_MESSAGE, ChatsTable.COLUMN_NAME_GROUPID, ChatsTable.COLUMN_NAME_IMAGE, ChatsTable.COLUMN_NAME_DATE, ChatsTable.COLUMN_NAME_ID)
        var selection = "${ChatsTable.COLUMN_NAME_GROUPID} = ?"
        var selectionArgs = arrayOf(groupId)

        var cursor = db.query(ChatsTable.TABLE_NAME,
        projection,
        selection,
        selectionArgs,
        null,
        null,
        null)

        var lst: ArrayList<Chat> = ArrayList()

        while (cursor.moveToNext()){
            var chat: Chat = Chat()
            chat.chatId = cursor.getString(cursor.getColumnIndexOrThrow(ChatsTable.COLUMN_NAME_CHAT_ID))
            chat.date = cursor.getString(cursor.getColumnIndexOrThrow(ChatsTable.COLUMN_NAME_DATE))
            chat.groupId = cursor.getString(cursor.getColumnIndexOrThrow(ChatsTable.COLUMN_NAME_GROUPID))
            chat.message = cursor.getString(cursor.getColumnIndexOrThrow(ChatsTable.COLUMN_NAME_MESSAGE))
            chat.time = cursor.getString(cursor.getColumnIndexOrThrow(ChatsTable.COLUMN_NAME_TIME))
            chat.userId = cursor.getString(cursor.getColumnIndexOrThrow(ChatsTable.COLUMN_NAME_USERID))
            chat.img = cursor.getString(cursor.getColumnIndexOrThrow(ChatsTable.COLUMN_NAME_IMAGE))

            lst.add(chat)
        }

        return lst
    }

    //trial
    suspend fun getTrialChats(groupId: String): MutableLiveData<List<Chat>>{
        var db = usersSqliteDatabaseHelper!!.readableDatabase
        var projection = arrayOf(ChatsTable.COLUMN_NAME_CHAT_ID, ChatsTable.COLUMN_NAME_USERID, ChatsTable.COLUMN_NAME_TIME,
            ChatsTable.COLUMN_NAME_MESSAGE, ChatsTable.COLUMN_NAME_GROUPID, ChatsTable.COLUMN_NAME_DATE, ChatsTable.COLUMN_NAME_ID)
        var selection = "${ChatsTable.COLUMN_NAME_GROUPID} = ?"
        var selectionArgs = arrayOf(groupId)

        var cursor = db.query(ChatsTable.TABLE_NAME,
            projection,
            selection,
            selectionArgs,
            null,
            null,
            null)

        var lst: ArrayList<Chat> = ArrayList()
        var ls: MutableLiveData<List<Chat>> = MutableLiveData()

        while (cursor.moveToNext()){
            var chat: Chat = Chat()
            chat.chatId = cursor.getString(cursor.getColumnIndexOrThrow(ChatsTable.COLUMN_NAME_CHAT_ID))
            chat.date = cursor.getString(cursor.getColumnIndexOrThrow(ChatsTable.COLUMN_NAME_DATE))
            chat.groupId = cursor.getString(cursor.getColumnIndexOrThrow(ChatsTable.COLUMN_NAME_GROUPID))
            chat.message = cursor.getString(cursor.getColumnIndexOrThrow(ChatsTable.COLUMN_NAME_MESSAGE))
            chat.time = cursor.getString(cursor.getColumnIndexOrThrow(ChatsTable.COLUMN_NAME_TIME))
            chat.userId = cursor.getString(cursor.getColumnIndexOrThrow(ChatsTable.COLUMN_NAME_USERID))

            lst.add(chat)
        }

        ls.value = lst

        return ls
    }

}

package com.example.collegecommapp.reporitories

import android.content.ContentValues
import android.content.Context
import android.database.sqlite.SQLiteDatabase
import android.util.Log
import com.example.collegecommapp.sqlitedatabase.helpers.UsersSqliteDatabaseHelper
import com.example.collegecommapp.sqlitedatabase.tables.GroupTable
import com.example.collegecommapp.models.Group


class GroupRepo(context: Context) {
    private val TAG = "GroupRepo"
    private var usersSqliteDatabaseHelper: UsersSqliteDatabaseHelper? = null

    init {
        usersSqliteDatabaseHelper = UsersSqliteDatabaseHelper(context)
    }

    companion object{
        private var INSTANCE: GroupRepo? = null

        fun setInstance(context: Context): GroupRepo{
            if (INSTANCE == null){
                INSTANCE = GroupRepo(context)
            }

            return INSTANCE!!
        }
    }

    //GROUPS
    //create group
    suspend fun createGroup(group: Group): Long{
        if (group.group_id != null){
            var db: SQLiteDatabase = usersSqliteDatabaseHelper!!.writableDatabase
            var contentValues: ContentValues = ContentValues().apply {
                put(GroupTable.COLUMN_NAME_GROUP_ID, group.group_id)
                put(GroupTable.COLUMN_NAME_GROUP_NAME, group.group_name)
                put(GroupTable.COLUMN_NAME_DESCRIPTION, group.group_description)
                put(GroupTable.COLUMN_NAME_CAPACITY, group.group_capacity)
                put(GroupTable.COLUMN_NAME_IMAGE, group.group_image)
                put(GroupTable.COLUMN_NAME_CREATEDBY, group.group_created_by)
                put(GroupTable.COLUMN_NAME_DATE_CREATED, group.group_date_created)
            }

            var response = db.insert(GroupTable.TABLE_NAME, null, contentValues)

            if (response >= 0){
                return response
            }
            else{
                return -1
            }
        }
        else{
            return -1
        }

    }

    suspend fun generateGroups(): ArrayList<Group>{
        var db: SQLiteDatabase = usersSqliteDatabaseHelper!!.readableDatabase
        var projection = arrayOf(
            GroupTable.COLUMN_NAME_ID, GroupTable.COLUMN_NAME_DATE_CREATED,
            GroupTable.COLUMN_NAME_CREATEDBY, GroupTable.COLUMN_NAME_IMAGE,
            GroupTable.COLUMN_NAME_CAPACITY, GroupTable.COLUMN_NAME_DESCRIPTION, GroupTable.COLUMN_NAME_GROUP_NAME, GroupTable.COLUMN_NAME_GROUP_ID)

        var cursor = db.query(
            GroupTable.TABLE_NAME,
            projection,
            null,
            null,
            null,
            null,
            null
        )

        var lst: ArrayList<Group> = ArrayList()

        while (cursor.moveToNext()){
            var group: Group = Group()
            group.group_id = cursor.getString(cursor.getColumnIndexOrThrow(GroupTable.COLUMN_NAME_GROUP_ID))
            group.group_name = cursor.getString(cursor.getColumnIndexOrThrow(GroupTable.COLUMN_NAME_GROUP_NAME))
            group.group_description = cursor.getString(cursor.getColumnIndexOrThrow(GroupTable.COLUMN_NAME_DESCRIPTION))
            group.group_image = cursor.getString(cursor.getColumnIndexOrThrow(GroupTable.COLUMN_NAME_IMAGE))
            group.group_capacity = cursor.getInt(cursor.getColumnIndexOrThrow(GroupTable.COLUMN_NAME_CAPACITY))
            group.group_date_created = cursor.getString(cursor.getColumnIndexOrThrow(GroupTable.COLUMN_NAME_DATE_CREATED))
            group.group_created_by = cursor.getString(cursor.getColumnIndexOrThrow(GroupTable.COLUMN_NAME_CREATEDBY))

            lst.add(group)
        }

        Log.i(TAG, "generateGroups: $lst")

        return lst
    }

    suspend fun getGroupDetails(id: String): Group {
        var db = usersSqliteDatabaseHelper!!.readableDatabase
        var projection = arrayOf(
            GroupTable.COLUMN_NAME_ID, GroupTable.COLUMN_NAME_DATE_CREATED,
            GroupTable.COLUMN_NAME_CREATEDBY, GroupTable.COLUMN_NAME_IMAGE,
            GroupTable.COLUMN_NAME_CAPACITY, GroupTable.COLUMN_NAME_DESCRIPTION, GroupTable.COLUMN_NAME_GROUP_NAME, GroupTable.COLUMN_NAME_GROUP_ID)

        var selection = "${GroupTable.COLUMN_NAME_GROUP_ID} = ?"
        var selectionArgs = arrayOf(id)

        var cursor = db.query(
            GroupTable.TABLE_NAME,
            projection,
            selection,
            selectionArgs,
            null,
            null,
            null
        )

        var group: Group = Group()

        while (cursor.moveToNext()){
            group.group_id = cursor.getString(cursor.getColumnIndexOrThrow(GroupTable.COLUMN_NAME_GROUP_ID))
            group.group_name = cursor.getString(cursor.getColumnIndexOrThrow(GroupTable.COLUMN_NAME_GROUP_NAME))
            group.group_description = cursor.getString(cursor.getColumnIndexOrThrow(GroupTable.COLUMN_NAME_DESCRIPTION))
            group.group_image = cursor.getString(cursor.getColumnIndexOrThrow(GroupTable.COLUMN_NAME_IMAGE))
            group.group_capacity = cursor.getInt(cursor.getColumnIndexOrThrow(GroupTable.COLUMN_NAME_CAPACITY))
            group.group_date_created = cursor.getString(cursor.getColumnIndexOrThrow(GroupTable.COLUMN_NAME_DATE_CREATED))
            group.group_created_by = cursor.getString(cursor.getColumnIndexOrThrow(GroupTable.COLUMN_NAME_CREATEDBY))

        }

        return group
    }
}
package com.example.collegecommapp.reporitories

import android.content.ContentValues
import android.content.Context
import android.database.sqlite.SQLiteDatabase
import android.util.Log
import com.example.collegecommapp.models.Member
import com.example.collegecommapp.sqlitedatabase.helpers.UsersSqliteDatabaseHelper
import com.example.collegecommapp.sqlitedatabase.queries.MembersQueries
import com.example.collegecommapp.sqlitedatabase.tables.MembersTable

class MembersRepository(context: Context) {
    private val TAG = "MembersRepository"
    private var usersSqliteDatabaseHelper: UsersSqliteDatabaseHelper? = null

    init {
        usersSqliteDatabaseHelper = UsersSqliteDatabaseHelper(context)
    }

    companion object{
        private var INSTANCE: MembersRepository? = null

        fun setInstance(context: Context): MembersRepository{
            if (INSTANCE == null){
                INSTANCE = MembersRepository(context)
            }

            return INSTANCE!!
        }
    }

    //MEMBERS
    //add member
    suspend fun addMember(member: Member): Long{
        var db: SQLiteDatabase = usersSqliteDatabaseHelper!!.writableDatabase
        var contentValues: ContentValues = ContentValues().apply {
            put(MembersTable.COLUMN_NAME_USERID, member.userId)
            put(MembersTable.COLUMN_NAME_GROUPID, member.groupId)
            put(MembersTable.COLUMN_NAME_CODE, member.code)
            put(MembersTable.COLUMN_NAME_DATE_ADDED, member.dateAdded)
        }

        var response = db.insert(MembersTable.TABLE_NAME, null, contentValues)

        Log.i(TAG, "addMember: ${response}")

        if (response >= 0){
            return response
        }
        else{
            return -1
        }

    }

    suspend fun getMembersByGroup(groupId: String): ArrayList<Member>{
        var db = usersSqliteDatabaseHelper!!.readableDatabase
        var projection =  arrayOf(
            MembersTable.COLUMN_NAME_ID, MembersTable.COLUMN_NAME_DATE_ADDED, MembersTable.COLUMN_NAME_USERID,
            MembersTable.COLUMN_NAME_GROUPID, MembersTable.COLUMN_NAME_CODE)
        var selection = "${MembersTable.COLUMN_NAME_GROUPID} = ?"
        var selectionArgs = arrayOf(groupId)

        var cursor = db.query(
            MembersTable.TABLE_NAME,
            projection,
            selection,
            selectionArgs,
            null,
            null,
            null
        )

        var lst: ArrayList<Member> = ArrayList()

        while (cursor.moveToNext()){
            var member: Member = Member()
            member.userId = cursor.getString(cursor.getColumnIndexOrThrow(MembersTable.COLUMN_NAME_USERID))
            member.groupId = cursor.getString(cursor.getColumnIndexOrThrow(MembersTable.COLUMN_NAME_GROUPID))
            member.code = cursor.getString(cursor.getColumnIndexOrThrow(MembersTable.COLUMN_NAME_CODE))
            member.dateAdded = cursor.getString(cursor.getColumnIndexOrThrow(MembersTable.COLUMN_NAME_DATE_ADDED))

            lst.add(member)
        }

        return lst
    }

    suspend fun getMemberGroups(userId: String): ArrayList<String>{
        var db = usersSqliteDatabaseHelper!!.readableDatabase
        var projection =  arrayOf(
            MembersTable.COLUMN_NAME_ID, MembersTable.COLUMN_NAME_DATE_ADDED, MembersTable.COLUMN_NAME_USERID,
            MembersTable.COLUMN_NAME_GROUPID, MembersTable.COLUMN_NAME_CODE)
        var selection = "${MembersTable.COLUMN_NAME_USERID} = ?"
        var selectionArgs = arrayOf(userId)

        var cursor = db.query(
            MembersTable.TABLE_NAME,
            projection,
            selection,
            selectionArgs,
            null,
            null,
            null
        )

        var lst: ArrayList<String> = ArrayList()

        while (cursor.moveToNext()){
            lst.add(cursor.getString(cursor.getColumnIndexOrThrow(MembersTable.COLUMN_NAME_GROUPID)))
        }

        return lst
    }

    suspend fun leaveGroup(userId: String, groupId: String): Int{
        var db = usersSqliteDatabaseHelper!!.writableDatabase
        var selection = "${MembersTable.COLUMN_NAME_USERID} = ? AND ${MembersTable.COLUMN_NAME_GROUPID} = ?"
        var selectionArgs = arrayOf(userId, groupId)
        var res = db.delete(MembersTable.TABLE_NAME,
            selection,
            selectionArgs
        )

        if (res >= 0){
            return res
        }
        return 0
    }

}
//here is model of the database 
package com.example.collegecommapp.models

class Chat(
    var chatId: String? = null,
    var userId: String? = null,
    var groupId: String? = null,
    var message: String? = null,
    var date: String? = null,
    var time: String? = null,
    var img: String? = null
) {
}
package com.example.collegecommapp.models

class Group(
    var id: Int? = null,
    var group_id: String? = null,
    var group_name: String? = null,
    var group_description: String? = null,
    var group_capacity: Int? = null,
    var group_image: String? = null,
    var group_created_by: String? = null,
    var group_date_created: String? = null
) {
}
package com.example.collegecommapp.models

class GroupDisplay(
    var group_id: String? = null,
    var group_name: String? = null,
    var group_image: String? = null,
    var message: String? = null,
    var date: String? = null,
    var time: String? = null,
    var total: String? = null
) {
}
package com.example.collegecommapp.models

class Member(
    var userId: String? = null,
    var groupId: String? = null,
    var code: String? = null,
    var dateAdded: String? = null
) {
}
package com.example.collegecommapp.models

class User(
    var userId: String? = null,
    var firstName: String? = null,
    var lastName: String? = null,
    var email: String? = null,
    var phone: String? = null,
    var password: String? = null
) {
}
//interface 
package com.example.collegecommapp.interfaces

import com.example.collegecommapp.models.Group

interface Generalinterface {

    fun logOut()
    fun addChatRoom(group: Group)
    fun goToChatPage(groupId: String)
    fun getPhoneDetails(userPhone: String): String?
    fun goToNewChatRooms()
    fun goToSearch()
    fun goToMainPage()
    fun goToProfile()

}

// chatclasses 
package com.example.collegecommapp.chatclasses

import android.content.ContentValues
import android.content.Context
import android.content.Intent
import android.content.SharedPreferences
import android.database.sqlite.SQLiteDatabase
import android.graphics.Bitmap
import android.graphics.ImageDecoder
import android.net.Uri
import android.os.Build
import android.os.Bundle
import android.text.TextUtils
import android.util.Log
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.view.Window
import android.widget.*
import androidx.activity.result.ActivityResultCallback
import androidx.activity.result.contract.ActivityResultContracts
import androidx.annotation.RequiresApi
import androidx.appcompat.app.AppCompatActivity
import androidx.fragment.app.Fragment
import androidx.lifecycle.Observer
import androidx.lifecycle.ViewModelProvider
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView
import com.example.collegecommapp.R
import com.example.collegecommapp.adapters.ChatAdapter
import com.example.collegecommapp.interfaces.Generalinterface
import com.example.collegecommapp.models.Chat
import com.example.collegecommapp.sqlitedatabase.helpers.UsersSqliteDatabaseHelper
import com.example.collegecommapp.sqlitedatabase.queries.ChatEntries
import com.example.collegecommapp.sqlitedatabase.queries.ChatEntries.Companion.CHAT_SQL_ENTRIES
import com.example.collegecommapp.sqlitedatabase.tables.ChatsTable
import com.example.collegecommapp.viewmodels.ChattingViewModel
import com.google.android.material.bottomsheet.BottomSheetDialog
import java.io.IOException
import java.text.SimpleDateFormat
import java.util.*
import kotlin.random.Random

class ChatRoom : Fragment(), View.OnClickListener {
    private val TAG = "ChatRoom"
    private lateinit var chat: EditText
    private lateinit var send: ImageView
    private lateinit var attach: ImageView
    private lateinit var bot: RelativeLayout
    private lateinit var recyclerView: RecyclerView
    private lateinit var desc: TextView
    private lateinit var linearLayoutManager: LinearLayoutManager
    private lateinit var back: RelativeLayout
    private lateinit var title: TextView
    private lateinit var imgChat: ImageView
    private lateinit var chattingViewModel: ChattingViewModel
    private var groupId: String? = null
    private var userId: String? = null
    private var chatList: ArrayList<Chat> = ArrayList()
    private lateinit var generalinterface: Generalinterface
    private var filePath: Uri? = null
    private var imgUrl: String? = null
    var chatAdapter = ChatAdapter(activity as Context)

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

    }

    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        // Inflate the layout for this fragment
        return inflater.inflate(R.layout.fragment_chat_room, container, false)
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        chattingViewModel = ViewModelProvider(requireActivity()).get(ChattingViewModel::class.java)


        recyclerView = view.findViewById(R.id.recyclerChats)
        chat = view.findViewById(R.id.editChat)
        attach = view.findViewById(R.id.attach)
        send = view.findViewById(R.id.send)
        back = view.findViewById(R.id.relBackChats)
        title = view.findViewById(R.id.txtTitleChat)
        desc = view.findViewById(R.id.txtDesc)
        bot = view.findViewById(R.id.relBot)
        imgChat = view.findViewById(R.id.imgChat)
        imgChat.visibility = View.GONE

        linearLayoutManager = LinearLayoutManager(activity)
        var chatAdapter = ChatAdapter(activity as Context)

        attach.setOnClickListener(this)
        send.setOnClickListener(this)
        back.setOnClickListener(this)
        bot.setOnClickListener(this)

        recyclerView.adapter = chatAdapter
        recyclerView.layoutManager = linearLayoutManager

        getGroupDetails()

        getChats()

        getUserId()
    }

    @RequiresApi(Build.VERSION_CODES.P)
    var chatPic = registerForActivityResult(ActivityResultContracts.StartActivityForResult(), ActivityResultCallback {
        if (it.resultCode == AppCompatActivity.RESULT_OK && it.data != null){
            filePath = it.data!!.data!!
            try {
                val source = ImageDecoder.createSource(requireActivity().contentResolver,
                    filePath!!
                )
                val bitmap: Bitmap = ImageDecoder.decodeBitmap(source)
                if (bitmap != null){
                    imgChat.visibility= View.VISIBLE
                    imgChat.setImageBitmap(bitmap)

                    sendToDb()
                }

            }
            catch (e: IOException){
                e.printStackTrace()
            }
        }
    })

    private fun sendToDb() {
        if (!filePath!!.equals(null)){
            Log.i(TAG, "sendToDb: " + "Sent" + filePath)


        }
        else{
            Toast.makeText(activity, "Not sent", Toast.LENGTH_LONG).show()
        }
    }

    private fun getUserId() {
        var sharedPrefs: SharedPreferences = activity?.getSharedPreferences(getString(R.string.User),
            Context.MODE_PRIVATE
        )!!

        userId = sharedPrefs.getString(getString(R.string.id), "")
    }

    private fun getGroupDetails() {
        var sharedPrefs: SharedPreferences = activity?.getSharedPreferences("GROUPID",
            Context.MODE_PRIVATE
        )!!

        groupId = sharedPrefs.getString("groupId", "")

        chattingViewModel.getGroupDetails(groupId!!).observe(viewLifecycleOwner, Observer {
            if (it != null){
                title.text = it.group_name
                desc.text = it.group_description
            }
        })
    }

    private fun getChats() {
        chattingViewModel.getGroupChats(groupId!!).observe(viewLifecycleOwner, Observer {
            if (it.isNotEmpty()){
                setChats(it)
            }
        })
    }

    private fun setChats(it: List<Chat>?) {
        for (i in it!!){
            chatList.add(i)
        }

        chatAdapter.getData(chatList)
        recyclerView.scrollToPosition(chatList.size - 1)
    }

    override fun onClick(p0: View?) {
        when(p0!!.id){

            R.id.send -> {
                sendChat()
            }
            R.id.relBackChats -> {

            }
            R.id.relBot -> {
                showBottomSheet()
            }
        }

    }

    private fun sendChat() {
        Log.i(TAG, "sendChat: Sending")
        var date = SimpleDateFormat("yyyy-MM-dd").format(Date())
        var chatId: String = Random.nextInt(10, 1000).toString()
        var time = SimpleDateFormat("hh:mm").format(Date())
        var message = chat.text.toString().trim()

        if (!TextUtils.isEmpty(message)){
            var nChat: Chat = Chat()
            nChat.userId = userId
            nChat.groupId = groupId
            nChat.time = time
            nChat.date = date
            nChat.message = message
            nChat.chatId = chatId
            nChat.img = imgUrl ?: ""

            val db = UsersSqliteDatabaseHelper(requireContext()).writableDatabase
            val values = ContentValues()
            values.put(ChatsTable.COLUMN_NAME_USERID, chatId)
            values.put(ChatsTable.COLUMN_NAME_USERID, userId)
            values.put(ChatsTable.COLUMN_NAME_GROUPID, groupId)
            values.put(ChatsTable.COLUMN_NAME_TIME, time)
            values.put(ChatsTable.COLUMN_NAME_DATE, date)
            values.put(ChatsTable.COLUMN_NAME_MESSAGE, message)
            values.put(ChatsTable.COLUMN_NAME_IMAGE, imgUrl)

            val success = db.insert(ChatEntries.CHAT_SQL_ENTRIES, null, values)
            db.close()

            if (success >= 0){
                Log.i(TAG, "sendChat: Added")
                chatList.add(nChat)
                if (chatList.size > 0){
                    chatAdapter.addNewData(nChat, chatList.size - 1)
                    recyclerView.scrollToPosition(chatList.size - 1)
                }
            }
            else{
                Log.i(TAG, "sendChat: Not Added")
            }
        }
    }

    private fun showBottomSheet() {
        var context = activity as Context
        var bottomSheetDialog: BottomSheetDialog = BottomSheetDialog(context, R.style.SheetDialog)
        bottomSheetDialog.requestWindowFeature(Window.FEATURE_NO_TITLE)
        bottomSheetDialog.setContentView(R.layout.leave_sheet)

        var join = bottomSheetDialog.findViewById<LinearLayout>(R.id.linLeave)
        bottomSheetDialog.show()

        join!!.setOnClickListener {
            leaveGroup()
            bottomSheetDialog.hide()
        }
    }

    private fun leaveGroup() {
        if (userId != null){
            var res = chattingViewModel.leaveGroup(userId!!, groupId!!)

            if (res > 0){

            }
            else{
                Log.i(TAG, "leaveGroup: Left")
            }
        }
    }



    override fun onAttach(context: Context) {
        super.onAttach(context)
        generalinterface = context as Generalinterface
    }


}
package com.example.collegecommapp.chatclasses

import android.annotation.SuppressLint
import android.content.Context
import android.content.Intent
import android.content.SharedPreferences
import android.graphics.Bitmap
import android.graphics.ImageDecoder
import android.media.MediaCodec.MetricsConstants.MODE
import android.net.Uri
import android.os.Build
import android.os.Bundle
import android.util.Log
import androidx.fragment.app.Fragment
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.view.Window
import android.widget.*
import androidx.activity.result.ActivityResultCallback
import androidx.activity.result.contract.ActivityResultContracts
import androidx.annotation.RequiresApi
import androidx.appcompat.app.AppCompatActivity
import androidx.lifecycle.Observer
import androidx.lifecycle.ViewModelProvider
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView
import com.example.collegecommapp.R
import com.example.collegecommapp.interfaces.Generalinterface
import com.example.collegecommapp.models.Group
import com.example.collegecommapp.models.GroupDisplay
import com.example.collegecommapp.viewmodels.ChattingViewModel
import com.google.android.material.bottomsheet.BottomSheetDialog
import com.google.android.material.button.MaterialButton
import com.google.android.material.floatingactionbutton.FloatingActionButton
import com.google.android.material.textfield.TextInputEditText
import java.io.IOException
import java.text.SimpleDateFormat
import java.util.*
import kotlin.collections.ArrayList
import kotlin.random.Random

class Dashboard : Fragment(), View.OnClickListener {
    private val TAG = "Dashboard"
    private lateinit var more: RelativeLayout
    private lateinit var floatingActionButton: ImageButton
    private lateinit var homeTxt: TextView
    private lateinit var profile: RelativeLayout
    private lateinit var generalinterface: Generalinterface
    private lateinit var recyclerView: RecyclerView
    private lateinit var linearLayoutManager: LinearLayoutManager
    private lateinit var chattingViewModel: ChattingViewModel
     private lateinit var btnChart: Button
    private lateinit var progress: ProgressBar
    private var groups: ArrayList<GroupDisplay> = ArrayList()
    private lateinit var sharedPreferences: SharedPreferences
    private var userId: String? = null
    private var filePath: Uri? = null
    private var imgUrl: String = ""
    private lateinit var pic: ImageView


    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

    }

    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        // Inflate the layout for this fragment
        return inflater.inflate(R.layout.fragment_dashboard, container, false)
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        chattingViewModel = ViewModelProvider(requireActivity()).get(ChattingViewModel::class.java)


        var context = activity as Context

        //views
        more = view.findViewById(R.id.relMore)
        floatingActionButton = view.findViewById(R.id.floatDashboard)
        profile = view.findViewById(R.id.relProfile)
        recyclerView = view.findViewById(R.id.recyclerGroups)
        homeTxt = view.findViewById(R.id.txtHome)
        homeTxt.visibility = View.VISIBLE
        recyclerView.visibility = View.GONE

        //clicks
        profile.setOnClickListener(this)
        floatingActionButton.setOnClickListener(this)
        more.setOnClickListener(this)

        //Sharedpreferences
        sharedPreferences = activity?.getSharedPreferences(getString(R.string.User), Context.MODE_PRIVATE)!!
        userId = sharedPreferences.getString(getString(R.string.id), "")

        //layout managers
        linearLayoutManager = LinearLayoutManager(activity)

        //list data
        getListData()
    }

    private fun getListData() {
        chattingViewModel.getMemberGroups(userId!!).observe(viewLifecycleOwner, Observer {
            if (it!!.isNotEmpty()){
                groups.clear()
                showRecycler(it)
            }
            else{
                Toast.makeText(activity, "Not Found", Toast.LENGTH_LONG).show()
            }
        })
    }

    private fun showRecycler(it: List<GroupDisplay>?) {
        for (i in it!!){
            groups.add(i)
        }

        if (groups.size > 0){
            homeTxt.visibility = View.GONE
            recyclerView.visibility = View.VISIBLE
            recyclerView.layoutManager = linearLayoutManager
        }
    }

    override fun onClick(p0: View?) {
        when(p0!!.id){
            R.id.relMore -> {
                showBottomSheet()
            }
            R.id.floatDashboard -> {
                showChatRoomAdditionSheet()
            }
            R.id.relProfile -> {
            }
        }

    }

    @RequiresApi(Build.VERSION_CODES.P)
    var chatPic = registerForActivityResult(ActivityResultContracts.StartActivityForResult(), ActivityResultCallback {
        if (it.resultCode == AppCompatActivity.RESULT_OK && it.data != null){
            filePath = it.data!!.data!!
            try {
                val source = ImageDecoder.createSource(requireActivity().contentResolver,
                    filePath!!
                )
                val bitmap: Bitmap = ImageDecoder.decodeBitmap(source)
                pic.setImageBitmap(bitmap)
                if (bitmap != null){
                    sendToDb()
                }
            }
            catch (e: IOException){
                e.printStackTrace()
            }
        }
    })

    private fun sendToDb() {
        if (!filePath!!.equals(null)){
            Log.i(TAG, "sendToDb: " + "Sent" + filePath)

        }
        else{
            Toast.makeText(activity, "Not sent", Toast.LENGTH_LONG).show()
        }
    }

    @SuppressLint("SimpleDateFormat")
    private fun showChatRoomAdditionSheet() {
        var context = activity as Context
        var bottomSheetDialog: BottomSheetDialog = BottomSheetDialog(context, R.style.SheetDialog)
        bottomSheetDialog.requestWindowFeature(Window.FEATURE_NO_TITLE)
        bottomSheetDialog.setContentView(R.layout.newchatroom_bottom_sheet)

        var name: EditText = bottomSheetDialog.findViewById<TextInputEditText>(R.id.chatName)!!
        var desc: EditText = bottomSheetDialog.findViewById<TextInputEditText>(R.id.chatDescription)!!
        var cap: EditText = bottomSheetDialog.findViewById<TextInputEditText>(R.id.chatCapacity)!!
        btnChart = bottomSheetDialog.findViewById<MaterialButton>(R.id.btnChat)!!
        progress = bottomSheetDialog.findViewById<ProgressBar>(R.id.progressNew)!!
        var picClick: TextView = bottomSheetDialog.findViewById<TextView>(R.id.chatPic)!!
        pic = bottomSheetDialog.findViewById<ImageView>(R.id.imgChatRoomPic)!!

        progress.visibility = View.GONE
        bottomSheetDialog.show()

        picClick.setOnClickListener {
            btnChart.isEnabled = false
            progress.visibility = View.VISIBLE
            val intent: Intent = Intent()
            intent.type = "image/*"
            intent.action = Intent.ACTION_GET_CONTENT
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.P) {
                chatPic.launch(intent)
            }
        }

        btnChart.setOnClickListener {

                var time = SimpleDateFormat("yyyy-MM-dd").format(Date())

                var group: Group = Group()
                group.group_name = name.text.toString().trim()
                group.group_description = desc.text.toString().trim()
                group.group_capacity = cap.text.toString().toInt()
                group.group_created_by = userId
                group.group_image = imgUrl
                group.group_date_created = time
                group.group_id = "${userId.toString()}${Random.nextInt(100, 10000).toString()}"

                if (group != null){
                    var resp = chattingViewModel.createGroup(group)

                    Log.i(TAG, "showChatRoomAdditionSheet: ${resp}")

                    if (resp >= 0){
                        Toast.makeText(activity, "Group Created Successfully, wait for joining code", Toast.LENGTH_LONG).show()
                        generalinterface.addChatRoom(group)
                        bottomSheetDialog.hide()
                    }
                    else{
                        Toast.makeText(activity, "Group Not Created, try again", Toast.LENGTH_LONG).show()
                    }
                }
            }

                Toast.makeText(activity, "Connect to the internet for code generation", Toast.LENGTH_LONG).show()


    }

    private fun showBottomSheet() {
        var context = activity as Context
        var bottomSheetDialog: BottomSheetDialog = BottomSheetDialog(context, R.style.SheetDialog)
        bottomSheetDialog.requestWindowFeature(Window.FEATURE_NO_TITLE)
        bottomSheetDialog.setContentView(R.layout.options_bottom_sheet)

        var join = bottomSheetDialog.findViewById<LinearLayout>(R.id.linJoin)
        var search = bottomSheetDialog.findViewById<LinearLayout>(R.id.linSearch)
        bottomSheetDialog.show()

        join!!.setOnClickListener {
            generalinterface.goToNewChatRooms()
            bottomSheetDialog.hide()
        }

        search!!.setOnClickListener {
            generalinterface.goToSearch()
            bottomSheetDialog.hide()
        }
    }

    override fun onAttach(context: Context) {
        super.onAttach(context)
        generalinterface = context as Generalinterface
    }
}
package com.example.collegecommapp.chatclasses

import android.content.Context
import android.content.SharedPreferences
import android.os.Bundle
import android.util.Log
import androidx.fragment.app.Fragment
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.RelativeLayout
import android.widget.TextView
import android.widget.Toast
import androidx.lifecycle.Observer
import androidx.lifecycle.ViewModelProvider
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView
import com.example.collegecommapp.R
import com.example.collegecommapp.adapters.ChatRoomsAdapter
import com.example.collegecommapp.interfaces.Generalinterface
import com.example.collegecommapp.models.Group
import com.example.collegecommapp.models.GroupDisplay
import com.example.collegecommapp.viewmodels.ChattingViewModel

class NewChatRooms : Fragment(), View.OnClickListener {
    private val TAG = "NewChatRooms"
    private lateinit var recyclerView: RecyclerView
    private lateinit var linearLayoutManager: LinearLayoutManager
    private lateinit var chatRoomsAdapter: ChatRoomsAdapter
    private lateinit var chattingViewModel: ChattingViewModel
    private lateinit var back: RelativeLayout
    private lateinit var search: RelativeLayout
    private lateinit var txtAva: TextView
    private lateinit var generalinterface: Generalinterface
    private lateinit var sharedPreferences: SharedPreferences
    private var newLst: ArrayList<Group>? = null
    private var userId: String? = null

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        chattingViewModel = ViewModelProvider(this)[ChattingViewModel::class.java]
    }

    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        // Inflate the layout for this fragment
        return inflater.inflate(R.layout.fragment_new_chat_rooms, container, false)
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        recyclerView = view.findViewById(R.id.recyclerNewChatRoom)
        back = view.findViewById(R.id.relBackNewChat)
        search = view.findViewById(R.id.relSearch)
        txtAva = view.findViewById(R.id.txtAvailable)
        linearLayoutManager = LinearLayoutManager(activity)
        chatRoomsAdapter = ChatRoomsAdapter(activity as Context)
        txtAva.visibility = View.VISIBLE

        back.setOnClickListener(this)
        search.setOnClickListener(this)

        //Sharedprefences
        sharedPreferences = activity?.getSharedPreferences(getString(R.string.User), Context.MODE_PRIVATE)!!
        userId = sharedPreferences.getString(getString(R.string.id), "")

        chattingViewModel.getGroups(userId!!).observe(viewLifecycleOwner, Observer {
            if (it!!.size > 0){
                showRecycler(it)
            }
            else{

            }
        })
    }

    private fun showRecycler(it: List<Group>?) {
        var arr: ArrayList<Group> = ArrayList()

        for (i in it!!){
            arr.add(i)
        }

        newLst = arr

        if (newLst!!.size > 0){
            txtAva.visibility = View.GONE
            chatRoomsAdapter.getData(newLst!!)
            recyclerView.adapter = chatRoomsAdapter
            recyclerView.layoutManager = linearLayoutManager
        }
    }

    override fun onClick(p0: View?) {
        when(p0!!.id){
            R.id.relBackNewChat -> {
                generalinterface.goToMainPage()
            }
            R.id.relSearch -> {
                generalinterface.goToSearch()
            }
        }
    }

    override fun onAttach(context: Context) {
        super.onAttach(context)
        generalinterface = context as Generalinterface
    }
}
package com.example.collegecommapp.chatclasses

import android.content.Context
import android.os.Bundle
import androidx.fragment.app.Fragment
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.RelativeLayout
import android.widget.TextView
import androidx.appcompat.app.AppCompatActivity
import com.example.collegecommapp.interfaces.Generalinterface
import com.example.collegecommapp.R

class Profile : Fragment(), View.OnClickListener {
    private lateinit var generalinterface: Generalinterface
    private lateinit var relLog: RelativeLayout
    private lateinit var back: RelativeLayout
    private lateinit var firstName: TextView
    private lateinit var lastName: TextView
    private lateinit var email: TextView
    private lateinit var phone: TextView

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

    }

    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        // Inflate the layout for this fragment
        return inflater.inflate(R.layout.fragment_profile, container, false)
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        relLog = view.findViewById(R.id.logOut)
        back = view.findViewById(R.id.relBackProf)
        firstName = view.findViewById(R.id.firstN)
        lastName = view.findViewById(R.id.lastN)
        email = view.findViewById(R.id.em)
        phone = view.findViewById(R.id.phoneProf)
        relLog.setOnClickListener(this)
        back.setOnClickListener(this)

        setProfile()
    }

    private fun setProfile() {
        var sharedPreferences = activity?.getSharedPreferences(getString(R.string.User),
            Context.MODE_PRIVATE
        )
        firstName.text = sharedPreferences!!.getString(getString(R.string.firstName), "")
        lastName.text = sharedPreferences.getString(getString(R.string.lastName), "")
        email.text = sharedPreferences.getString(getString(R.string.email), "")
        phone.text = sharedPreferences.getString(getString(R.string.phone), "")

    }

    override fun onAttach(context: Context) {
        super.onAttach(context)
        generalinterface = context as Generalinterface
    }

    override fun onClick(p0: View?) {
        when(p0!!.id){
            R.id.logOut -> {
                generalinterface.logOut()
            }
            R.id.relBackProf -> {
                generalinterface.goToMainPage()
            }
        }
    }
}
package com.example.collegecommapp.chatclasses

import android.content.Context
import android.content.SharedPreferences
import android.os.Bundle
import android.text.Editable
import android.text.TextWatcher
import androidx.fragment.app.Fragment
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.EditText
import android.widget.RelativeLayout
import androidx.lifecycle.Observer
import androidx.lifecycle.ViewModelProvider
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView
import com.example.collegecommapp.R
import com.example.collegecommapp.adapters.ChatRoomsAdapter
import com.example.collegecommapp.adapters.GroupsAdapter
import com.example.collegecommapp.interfaces.Generalinterface
import com.example.collegecommapp.models.GroupDisplay
import com.example.collegecommapp.viewmodels.ChattingViewModel

class Search : Fragment(), View.OnClickListener {
    private lateinit var search: EditText
    private lateinit var back: RelativeLayout
    private lateinit var generalinterface: Generalinterface
    private lateinit var recyclerView: RecyclerView
    private lateinit var linearLayoutManager: LinearLayoutManager
    private lateinit var groupsAdapter: GroupsAdapter
    private lateinit var chattingViewModel: ChattingViewModel
    private lateinit var sharedPreferences: SharedPreferences
    private var userId: String? = null

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

    }

    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        // Inflate the layout for this fragment
        return inflater.inflate(R.layout.fragment_search, container, false)
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        chattingViewModel = ViewModelProvider(requireActivity()).get(ChattingViewModel::class.java)
        search = view.findViewById(R.id.editSearch)
        back = view.findViewById(R.id.relBackSearch)
        recyclerView = view.findViewById(R.id.recSearch)
        linearLayoutManager = LinearLayoutManager(activity)
        groupsAdapter = GroupsAdapter(activity as Context)

        back.setOnClickListener(this)

        //Sharedprefences
        sharedPreferences = activity?.getSharedPreferences(getString(R.string.User), Context.MODE_PRIVATE)!!
        userId = sharedPreferences.getString(getString(R.string.id), "")

        searchItems()

        getGroups()
    }

    private fun getGroups() {
        chattingViewModel.getMemberGroups(userId!!).observe(viewLifecycleOwner, Observer {
            if (it!!.isNotEmpty()){
                showRecycler(it)
            }
        })
    }

    private fun showRecycler(it: List<GroupDisplay>?) {
        groupsAdapter.getData(it as ArrayList<GroupDisplay>)
        recyclerView.adapter = groupsAdapter
        recyclerView.layoutManager = linearLayoutManager
    }

    private fun searchItems() {
        search.addTextChangedListener(object : TextWatcher{
            override fun beforeTextChanged(p0: CharSequence?, p1: Int, p2: Int, p3: Int) {

            }

            override fun onTextChanged(p0: CharSequence?, p1: Int, p2: Int, p3: Int) {
                groupsAdapter.filter.filter(p0)
            }

            override fun afterTextChanged(p0: Editable?) {

            }
        })
    }

    override fun onClick(p0: View?) {
        when(p0!!.id){
            R.id.relBackSearch -> {
                generalinterface.goToMainPage()
            }
        }
    }

    override fun onAttach(context: Context) {
        super.onAttach(context)
        generalinterface = context as Generalinterface
    }
}
package com.example.collegecommapp.chatclasses

import android.content.Context
import android.content.SharedPreferences
import android.os.Bundle
import android.text.Editable
import android.text.TextWatcher
import androidx.fragment.app.Fragment
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.EditText
import android.widget.TextView
import android.widget.Toast
import androidx.lifecycle.ViewModelProvider
import com.example.collegecommapp.R
import com.example.collegecommapp.interfaces.Generalinterface
import com.example.collegecommapp.models.Member
import com.example.collegecommapp.viewmodels.ChattingViewModel
import com.google.android.material.button.MaterialButton
import java.lang.StringBuilder
import java.text.SimpleDateFormat
import java.util.*

class Verification : Fragment(), View.OnClickListener {
    private lateinit var sharedPreferences: SharedPreferences
    private lateinit var phone: TextView
    private lateinit var numOne: EditText
    private lateinit var numTwo: EditText
    private lateinit var numThree: EditText
    private lateinit var numFour: EditText
    private lateinit var numFive: EditText
    private lateinit var numSix: EditText
    private lateinit var btn: MaterialButton
    private lateinit var resend: TextView
    private lateinit var chattingViewModel: ChattingViewModel
    private var code: String? = null
    private var groupId: String? = null
    private var user: String? = null
    private var userPhone: String? = null
    private lateinit var generalinterface: Generalinterface

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

    }

    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        // Inflate the layout for this fragment
        return inflater.inflate(R.layout.fragment_verification, container, false)
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        chattingViewModel = ViewModelProvider(requireActivity()).get(ChattingViewModel::class.java)

        phone = view.findViewById(R.id.phoneNumber)
        numOne = view.findViewById(R.id.editOne)
        numTwo = view.findViewById(R.id.editTwo)
        numThree = view.findViewById(R.id.editThree)
        numFour = view.findViewById(R.id.editFour)
        numFive = view.findViewById(R.id.editFive)
        numSix = view.findViewById(R.id.editSix)
        btn = view.findViewById(R.id.btnCode)
        resend = view.findViewById(R.id.resend)

        btn.setOnClickListener(this)
        resend.setOnClickListener(this)

        sharedPreferences = activity?.getSharedPreferences("CODE", Context.MODE_PRIVATE)!!
        code = sharedPreferences.getString("NUMBER", "")
        groupId = sharedPreferences.getString("GroupId", "")

        //user
        var sharedPreferences2 = activity?.getSharedPreferences("USER", Context.MODE_PRIVATE)
        user = sharedPreferences2!!.getString(getString(R.string.id), "")
        userPhone = sharedPreferences2.getString(getString(R.string.phone), "")

        phone.text = "+254${userPhone}"

        textChanging()
    }

    private fun textChanging() {
        numOne.addTextChangedListener(object : TextWatcher{
            override fun beforeTextChanged(p0: CharSequence?, p1: Int, p2: Int, p3: Int) {

            }

            override fun onTextChanged(p0: CharSequence?, p1: Int, p2: Int, p3: Int) {
                numTwo.requestFocus()
            }

            override fun afterTextChanged(p0: Editable?) {

            }

        } )
        numTwo.addTextChangedListener(object : TextWatcher{
            override fun beforeTextChanged(p0: CharSequence?, p1: Int, p2: Int, p3: Int) {

            }

            override fun onTextChanged(p0: CharSequence?, p1: Int, p2: Int, p3: Int) {
                numThree.requestFocus()
            }

            override fun afterTextChanged(p0: Editable?) {

            }

        } )
        numThree.addTextChangedListener(object : TextWatcher{
            override fun beforeTextChanged(p0: CharSequence?, p1: Int, p2: Int, p3: Int) {

            }

            override fun onTextChanged(p0: CharSequence?, p1: Int, p2: Int, p3: Int) {
                numFour.requestFocus()
            }

            override fun afterTextChanged(p0: Editable?) {

            }

        } )
        numFour.addTextChangedListener(object : TextWatcher{
            override fun beforeTextChanged(p0: CharSequence?, p1: Int, p2: Int, p3: Int) {

            }

            override fun onTextChanged(p0: CharSequence?, p1: Int, p2: Int, p3: Int) {
                numFour.requestFocus()
            }

            override fun afterTextChanged(p0: Editable?) {

            }

        } )
        numFour.addTextChangedListener(object : TextWatcher{
            override fun beforeTextChanged(p0: CharSequence?, p1: Int, p2: Int, p3: Int) {

            }

            override fun onTextChanged(p0: CharSequence?, p1: Int, p2: Int, p3: Int) {
                numFive.requestFocus()
            }

            override fun afterTextChanged(p0: Editable?) {

            }

        } )
        numFive.addTextChangedListener(object : TextWatcher{
            override fun beforeTextChanged(p0: CharSequence?, p1: Int, p2: Int, p3: Int) {

            }

            override fun onTextChanged(p0: CharSequence?, p1: Int, p2: Int, p3: Int) {
                numSix.requestFocus()
            }

            override fun afterTextChanged(p0: Editable?) {

            }

        } )
    }

    private fun sendDetails() {
        var member: Member = Member()
        member.groupId = groupId
        member.dateAdded = SimpleDateFormat("yyyy-MM-dd").format(Date())
        member.code = code
        member.userId = user

        var response = chattingViewModel.addMember(member)
        if (response >= 0){
            generalinterface.goToChatPage(groupId!!)
            Toast.makeText(activity, "Added", Toast.LENGTH_LONG).show()
        }
        else{
            Toast.makeText(activity, "Not Added", Toast.LENGTH_LONG).show()
        }
    }

    override fun onClick(p0: View?) {
        when(p0!!.id){
            R.id.btnCode -> {
                checkDetails()
            }
            R.id.resend -> {
                resendCode()
            }
        }
    }

    private fun resendCode() {
        var result = generalinterface.getPhoneDetails(userPhone!!)
        if (result != null){
            code = result
        }
    }

    private fun checkDetails() {
        var one = numOne.text.toString().trim()
        var two = numTwo.text.toString().trim()
        var three = numThree.text.toString().trim()
        var four = numFour.text.toString().trim()
        var five = numFive.text.toString().trim()
        var six = numSix.text.toString().trim()

        var stringBuilder: StringBuilder = StringBuilder()

        if (one != "" || two != "" || three != "" || four != "" || five != "" || six != ""){
            stringBuilder.append(one)
            stringBuilder.append(two)
            stringBuilder.append(three)
            stringBuilder.append(four)
            stringBuilder.append(five)
            stringBuilder.append(six)

            if (stringBuilder.toString() == code){
                sendDetails()
            }
            else{
                Toast.makeText(activity, "Invalid Code", Toast.LENGTH_LONG).show()
            }
        }
        else{
            Toast.makeText(activity, "Missing Number", Toast.LENGTH_LONG).show()
        }

    }

    override fun onAttach(context: Context) {
        super.onAttach(context)
        generalinterface = context as Generalinterface
    }
}
//authertication 
package com.example.collegecommapp.auth

import android.content.Intent
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import android.view.View
import com.example.collegecommapp.R
import com.example.collegecommapp.MainActivity
import com.google.android.material.button.MaterialButton

class CodeVerification : AppCompatActivity(), View.OnClickListener {
    private lateinit var btn: MaterialButton
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_codeverification)
        initViews()
    }

    private fun initViews() {
        btn = findViewById(R.id.btnVerify)
        btn.setOnClickListener(this)
    }

    override fun onClick(p0: View?) {
        when(p0!!.id){
            R.id.btnVerify -> {
                startActivity(Intent(this, MainActivity::class.java))
            }
        }
    }
}
package com.example.collegecommapp.auth

import android.content.Intent
import android.content.SharedPreferences
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import android.text.TextUtils
import android.util.Log
import android.view.View
import android.widget.Button
import android.widget.EditText
import android.widget.ImageView
import android.widget.TextView
import android.widget.Toast
import androidx.lifecycle.ViewModelProvider
import com.example.collegecommapp.MainActivity
import com.example.collegecommapp.R
import com.example.collegecommapp.viewmodels.RegisterActivityViewModel
import com.google.android.material.button.MaterialButton
import com.google.android.material.textfield.TextInputEditText

class Login : AppCompatActivity(), View.OnClickListener {
    private val TAG = "LoginUser"
    private lateinit var register: TextView
    private lateinit var email: EditText
    private lateinit var password: EditText
    private lateinit var btn: Button
    private lateinit var registerActivityViewModel: RegisterActivityViewModel
    private var sharedPreferences: SharedPreferences? = null
    private lateinit var back: ImageView

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_login)
        registerActivityViewModel = ViewModelProvider(this).get(RegisterActivityViewModel::class.java)
        initViews()
    }

    private fun initViews() {
        register = findViewById(R.id.txtRegister)
        email = findViewById(R.id.emailLogin)
        password = findViewById(R.id.passwordLogin)
        btn = findViewById(R.id.btnLogin)
        back = findViewById(R.id.loginBack)
        btn.setOnClickListener(this)
        register.setOnClickListener(this)
        back.setOnClickListener(this)
    }

    override fun onClick(p0: View?) {
        when(p0!!.id){
            R.id.txtRegister -> {
                startActivity(Intent(this, Register::class.java))
            }
            R.id.btnLogin -> {
                loginUser()
            }
            R.id.loginBack -> {
                finish()
            }
        }
    }

    private fun loginUser() {
        val em = email.text.toString().trim()
        val pass = password.text.toString().trim()

        if (TextUtils.isEmpty(em)){
            email.error = "Email Required"
            email.requestFocus()
        }
        else if (TextUtils.isEmpty(pass)){
            password.error = "Password Required"
            password.requestFocus()
        }
        else{
            btn.isEnabled = false
            var response = registerActivityViewModel.loginUser(em, pass)
            if (response != null){
                btn.isEnabled = true
                if (response.userId != null){
                    sharedPreferences = this.getSharedPreferences(getString(R.string.User), MODE_PRIVATE)
                    var editor: SharedPreferences.Editor = sharedPreferences!!.edit()
                    editor.putString(getString(R.string.firstName), response.firstName)
                    editor.putString(getString(R.string.lastName), response.lastName)
                    editor.putString(getString(R.string.email), response.email)
                    editor.putString(getString(R.string.phone), response.phone)
                    editor.putString(getString(R.string.id), response.userId)
                    editor.apply()
                    Toast.makeText(this, "Login successful", Toast.LENGTH_LONG).show()
                    goToMain()

                    Log.i(TAG, "loginUser: ${response.userId}")
                }
                else{
                    Toast.makeText(this, "Login Unsuccessful, check details", Toast.LENGTH_LONG).show()
                }
            }
            else{
                btn.isEnabled = true
                Toast.makeText(this, "Login Unsuccessful, check details", Toast.LENGTH_LONG).show()
            }

        }
    }

    private fun goToMain() {
        startActivity(Intent(this, MainActivity::class.java))
        finish()
    }

    override fun onStart() {
        super.onStart()
        sharedPreferences = this.getSharedPreferences(getString(R.string.User), MODE_PRIVATE)
        var userEmail = sharedPreferences!!.getString(getString(R.string.email), "")
        if (userEmail != null && userEmail != ""){
            startActivity(Intent(this, MainActivity::class.java))
            finish()
        }
    }
}
package com.example.collegecommapp.auth

import android.content.Intent
import android.content.SharedPreferences
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import android.text.TextUtils
import android.util.Log
import android.view.View
import android.widget.Button
import android.widget.EditText
import android.widget.ImageView
import android.widget.Toast
import androidx.lifecycle.ViewModelProvider
import com.example.collegecommapp.MainActivity
import com.example.collegecommapp.R
import com.example.collegecommapp.viewmodels.RegisterActivityViewModel
import com.google.android.material.button.MaterialButton
import com.google.android.material.textfield.TextInputEditText

class Register : AppCompatActivity(), View.OnClickListener {
    private val TAG = "Register"
    private lateinit var firstName: TextInputEditText
    private lateinit var lastName: TextInputEditText
    private lateinit var email: TextInputEditText
    private lateinit var password: TextInputEditText
    private lateinit var phone: TextInputEditText
    private lateinit var btn: MaterialButton
    private lateinit var back: ImageView
    private lateinit var registerActivityViewModel: RegisterActivityViewModel
    private lateinit var sharedPreferences: SharedPreferences

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_register)

        registerActivityViewModel = ViewModelProvider(this).get(RegisterActivityViewModel::class.java)
        initViews()

    }

    private fun initViews() {
        phone = findViewById(R.id.phone)
        firstName = findViewById(R.id.firstname)
        lastName = findViewById(R.id.lastname)
        email = findViewById(R.id.email)
        password = findViewById(R.id.password)
        btn = findViewById(R.id.btnRegister)
        back = findViewById(R.id.registerBack)
        back.setOnClickListener(this)
        btn.setOnClickListener(this)
    }

    override fun onClick(p0: View?) {
        when(p0!!.id){
            R.id.btnRegister -> {
                registerUser()
            }
            R.id.registerBack -> {
                startActivity(Intent(this, Login::class.java))
            }
        }
    }

    private fun registerUser() {
        val first = firstName.text.toString().trim()
        val last = lastName.text.toString().trim()
        val em = email.text.toString().trim()
        val pass = password.text.toString().trim()
        val pn = phone.text.toString().trim()
        if (TextUtils.isEmpty(pn)){
            phone.error = "Phone Required"
            phone.requestFocus()
        }
        else if (TextUtils.isEmpty(first)){
            firstName.error = "First Name Required"
            firstName.requestFocus()
        }
        else if (TextUtils.isEmpty(last)){
            lastName.error = "Last Name Required"
            lastName.requestFocus()
        }
        else if (TextUtils.isEmpty(em)){
            email.error = "Email Required"
            email.requestFocus()
        }
        else if (TextUtils.isEmpty(pass)){
            password.error = "Password Required"
            password.requestFocus()
        }
        else if (pass.length < 8){
            password.error = "Password Should have eight or more characters"
            password.requestFocus()
        }
    }

    private fun goToLogin() {
        startActivity(Intent(this, Login::class.java))
        finish()
    }

}
package com.example.collegecommapp.auth

import android.content.Intent
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import android.os.Handler
import android.os.Looper
import com.example.collegecommapp.R

class






SplashScreen : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_splashscreen)

        Handler(Looper.getMainLooper()).postDelayed({
            startActivity(Intent(this, Login::class.java))
            finish()
        }, 2000)

    }
}

// to check for the connection wifi and Celler
package com.example.collegecommapp.apputils

import android.content.Context
import android.net.ConnectivityManager
import android.net.NetworkCapabilities
import android.os.Build

class AppUtils(var context: Context) {

    fun checkWiFi(): Boolean{
        val connectivityManager: ConnectivityManager = context.getSystemService(Context.CONNECTIVITY_SERVICE) as ConnectivityManager

        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M){
            val network = connectivityManager.activeNetwork ?: return false
            val activeNetwork = connectivityManager.getNetworkCapabilities(network) ?: return false

            return when{
                activeNetwork.hasTransport(NetworkCapabilities.TRANSPORT_WIFI) -> true
                activeNetwork.hasTransport(NetworkCapabilities.TRANSPORT_CELLULAR) -> true

                else -> false
            }
        }
        else{
            @Suppress("DEPRECATION")
            val networkInfo = connectivityManager.activeNetworkInfo ?: return false
            @Suppress("DEPRECATION")
            return networkInfo.isConnected
        }

    }
}
//Adapters

package com.example.collegecommapp.adapters

import android.content.Context
import android.content.Context.MODE_PRIVATE
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.ImageView
import android.widget.TextView
import androidx.recyclerview.widget.RecyclerView
import com.example.collegecommapp.R
import com.example.collegecommapp.models.Chat
import com.squareup.picasso.OkHttp3Downloader
import com.squareup.picasso.Picasso

open class ChatAdapter(var context: Context): RecyclerView.Adapter<RecyclerView.ViewHolder>() {
    private var lst: ArrayList<Chat> = ArrayList()
    var sharedPreferences = context.getSharedPreferences(context.getString(R.string.User), MODE_PRIVATE)
    var userId = sharedPreferences!!.getString(context.getString(R.string.id), "")
    private val SEND_TYPE = userId!!.toInt()

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): RecyclerView.ViewHolder {
        if (viewType == SEND_TYPE){
            return SendViewModel(LayoutInflater.from(parent.context).inflate(R.layout.send_item, parent, false))
        }
        else{
            return ReceiveViewModel(LayoutInflater.from(parent.context).inflate(R.layout.receive_item, parent, false))
        }
    }

    override fun onBindViewHolder(holder: RecyclerView.ViewHolder, position: Int) {
        var chat: Chat = lst[position]
        if (chat.userId!!.toInt() == SEND_TYPE){
            var sendViewModel: SendViewModel = holder as SendViewModel
            sendViewModel.name.text = chat.userId.toString()
            sendViewModel.content.text = chat.message
            sendViewModel.time.text = chat.time
            sendViewModel.img.visibility = View.GONE
            if (chat.img != ""){
                sendViewModel.img.visibility = View.VISIBLE
                var picasso: Picasso.Builder = Picasso.Builder(context)
                picasso.downloader(OkHttp3Downloader(context))
                picasso.build().load(chat.img).into(sendViewModel.img)
            }
        }
        else{
            var receiveViewModel: ReceiveViewModel = holder as ReceiveViewModel
            receiveViewModel.recName.text = chat.userId.toString()
            receiveViewModel.recContent.text = chat.message
            receiveViewModel.recTime.text = chat.time
            receiveViewModel.img.visibility = View.GONE
            if (chat.img != ""){
                receiveViewModel.img.visibility = View.VISIBLE
                var picasso: Picasso.Builder = Picasso.Builder(context)
                picasso.downloader(OkHttp3Downloader(context))
                picasso.build().load(chat.img).into(receiveViewModel.img)
            }
        }
    }

    override fun getItemCount(): Int {
        return lst.size
    }

    override fun getItemViewType(position: Int): Int {
        return lst[position].userId!!.toInt()

    }

    protected class SendViewModel(view: View): RecyclerView.ViewHolder(view){
        var name: TextView = view.findViewById(R.id.nmChat)
        var content: TextView = view.findViewById(R.id.chatContent)
        var time: TextView = view.findViewById(R.id.chatTime)
        var img: ImageView = view.findViewById(R.id.imgChatMessage)
    }

    protected class ReceiveViewModel(view: View): RecyclerView.ViewHolder(view){
        var recName: TextView = view.findViewById(R.id.recName)
        var recContent: TextView = view.findViewById(R.id.recContent)
        var recTime: TextView = view.findViewById(R.id.recTime)
        var img: ImageView = view.findViewById(R.id.imgReceiveMessage)
    }

    fun getData(arr: ArrayList<Chat>){
        for (i in arr){
            lst.add(i)
            notifyDataSetChanged()
        }
    }

    fun addNewData(chat: Chat, pos: Int){
        lst.add(chat)
        notifyItemInserted(pos)
    }

}

package com.example.collegecommapp.adapters

import android.content.Context
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.ImageView
import android.widget.RelativeLayout
import android.widget.TextView
import androidx.recyclerview.widget.RecyclerView
import com.example.collegecommapp.R
import com.example.collegecommapp.interfaces.Generalinterface
import com.example.collegecommapp.models.Group
import com.squareup.picasso.OkHttp3Downloader
import com.squareup.picasso.Picasso


open class ChatRoomsAdapter(var context: Context): RecyclerView.Adapter<RecyclerView.ViewHolder>() {
    private var groupList: ArrayList<Group> = ArrayList()
    private lateinit var generalinterface: Generalinterface
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): RecyclerView.ViewHolder {
        return MyViewHolder(LayoutInflater.from(parent.context).inflate(R.layout.new_chat_item, parent, false))
    }

    override fun onBindViewHolder(holder: RecyclerView.ViewHolder, position: Int) {
        var myViewHolder: MyViewHolder = holder as MyViewHolder
        myViewHolder.title.text = groupList!![position].group_name
        myViewHolder.message.text = groupList!![position].group_description
        if (groupList[position].group_image != ""){
            var picasso: Picasso.Builder = Picasso.Builder(context)
            picasso.downloader(OkHttp3Downloader(context))
            picasso.build().load(groupList[position].group_image).into(myViewHolder.image)
        }

        myViewHolder.join.setOnClickListener {
            generalinterface.addChatRoom(groupList!![position])
        }
    }

    override fun getItemCount(): Int {
        return groupList!!.size
    }

    override fun onAttachedToRecyclerView(recyclerView: RecyclerView) {
        super.onAttachedToRecyclerView(recyclerView)
        generalinterface = context as Generalinterface
    }

    protected class MyViewHolder(view: View): RecyclerView.ViewHolder(view){
        var image: ImageView = view.findViewById(R.id.imgChatImage)
        var title: TextView = view.findViewById(R.id.groupChatName)
        var message: TextView = view.findViewById(R.id.groupChatDescription)
        var join: RelativeLayout = view.findViewById(R.id.relJoin)
    }

    fun getData(lst: ArrayList<Group>){
        for (i in lst){
            groupList!!.add(i)
            notifyDataSetChanged()
        }
    }
}

package com.example.collegecommapp.adapters

import android.content.Context
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.*
import androidx.recyclerview.widget.RecyclerView
import com.example.collegecommapp.R
import com.example.collegecommapp.interfaces.Generalinterface
import com.example.collegecommapp.models.GroupDisplay
import com.squareup.picasso.OkHttp3Downloader
import com.squareup.picasso.Picasso

open class GroupsAdapter(var context: Context): RecyclerView.Adapter<RecyclerView.ViewHolder>(), Filterable {
    private var lst: ArrayList<GroupDisplay> = ArrayList()
    private var filteredList: ArrayList<GroupDisplay> = ArrayList()
    private lateinit var generalinterface: Generalinterface

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): RecyclerView.ViewHolder {
        return MyViewHolder(LayoutInflater.from(parent.context).inflate(R.layout.group_item, parent, false))
    }

    override fun onBindViewHolder(holder: RecyclerView.ViewHolder, position: Int) {
        var myViewHolder: MyViewHolder = holder as MyViewHolder
        myViewHolder.title.text = lst!![position].group_name
        myViewHolder.message.text = lst[position].message ?: ""
        myViewHolder.time.text = lst[position].time
        myViewHolder.total.text = lst[position].total
        if (lst[position].group_image != ""){
            var picasso: Picasso.Builder = Picasso.Builder(context)
            picasso.downloader(OkHttp3Downloader(context))
            picasso.build().load(lst[position].group_image).into(myViewHolder.image)
        }

        myViewHolder.relCont.setOnClickListener {
            generalinterface.goToChatPage(lst[position].group_id!!)
        }
    }

    override fun getItemCount(): Int {
        return lst!!.size
    }

    override fun onAttachedToRecyclerView(recyclerView: RecyclerView) {
        super.onAttachedToRecyclerView(recyclerView)
        generalinterface = context as Generalinterface
    }

    protected class MyViewHolder(view: View): RecyclerView.ViewHolder(view) {
        var image: ImageView = view.findViewById(R.id.imgGroupImage)
        var title: TextView = view.findViewById(R.id.groupName)
        var message: TextView = view.findViewById(R.id.groupMessage)
        var time: TextView = view.findViewById(R.id.groupTime)
        var total: TextView = view.findViewById(R.id.groupTotal)
        var relCont: RelativeLayout = view.findViewById(R.id.groupId)

    }

    fun getData(data: ArrayList<GroupDisplay>){
        for (i in data){
            lst!!.add(i)
            notifyDataSetChanged()
        }

        filteredList.addAll(data)
        notifyDataSetChanged()
    }

    override fun getFilter(): Filter {
        return object: Filter(){
            override fun performFiltering(p0: CharSequence?): FilterResults {
                var filList: ArrayList<GroupDisplay> = ArrayList()
                if (p0.isNullOrEmpty()){
                    filList.addAll(filteredList)
                }
                else{
                    var name = p0.toString().lowercase().trim()
                    for (i in filteredList){
                        if (i.group_name.toString().lowercase().contains(name)){
                            filList.add(i)
                        }
                    }
                }
                var result: FilterResults = FilterResults()
                result.values = filList
                return result
            }

            override fun publishResults(p0: CharSequence?, p1: FilterResults?) {
                lst.clear()
                lst.addAll(p1!!.values as ArrayList<GroupDisplay>)
                notifyDataSetChanged()
            }

        }
    }
}
// now here is the layout of xml 
//Activity_main 
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <fragment
        android:id="@+id/navFrag"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:name="androidx.navigation.fragment.NavHostFragment"
        app:defaultNavHost="true"
        app:navGraph="@navigation/nav_graph"
        />

</androidx.constraintlayout.widget.ConstraintLayout>
//Acvtivity_login 
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".auth.Login">

    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="56dp"

        >
        <LinearLayout
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            android:layout_centerVertical="true"
            android:layout_marginStart="18dp"
            tools:ignore="UselessParent">
            <ImageView
                android:id="@+id/loginBack"
                android:layout_width="30dp"
                android:layout_height="30dp"
                android:src="@drawable/backarrow"
                android:scaleType="fitXY"
                android:layout_gravity="center_vertical"
                app:tint="@color/primary"
                android:contentDescription="TODO" />
            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@string/login"
                android:fontFamily="@font/robotobold"
                android:textColor="@color/primary"
                android:layout_gravity="center_vertical"
                android:textSize="20sp"
                android:layout_marginStart="10dp"
                />

        </LinearLayout>

    </RelativeLayout>
    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        >
        <LinearLayout
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:layout_margin="20dp"
            >
            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@string/welcome"
                android:fontFamily="@font/robotoregular"
                android:textStyle="bold"
                android:textSize="20sp"
                android:textColor="@color/primary"
                />
            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Please login to continue"
                android:fontFamily="@font/robotoregular"
                android:layout_marginTop="10dp"
                android:textSize="16sp"
                android:textColor="@android:color/darker_gray"
                />

        </LinearLayout>
    </RelativeLayout>
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        >
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginStart="20dp"
            android:layout_marginTop="10dp"
            android:layout_marginEnd="20dp"
            android:hint="Email Address"
            app:boxStrokeColor="@color/black"
            app:boxStrokeWidth="0.5dp"
            app:hintTextColor="@android:color/darker_gray"
            app:startIconDrawable="@drawable/emailnew"/>

        <EditText
            android:id="@+id/emailLogin"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:inputType="textEmailAddress"
            android:minHeight="48dp"
            tools:ignore="VisualLintTextFieldSize" />


        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginStart="20dp"
            android:layout_marginTop="10dp"
            android:layout_marginEnd="20dp"
            android:hint="Password"
            app:boxStrokeColor="@color/black"
            app:hintTextColor="@android:color/darker_gray"
            app:passwordToggleEnabled="true"
            app:boxStrokeWidth="0.5dp"
            app:startIconDrawable="@drawable/locknew"/>

        <EditText
            android:id="@+id/passwordLogin"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:inputType="textPassword"
            android:minHeight="48dp"
            tools:ignore="VisualLintTextFieldSize" />

        <RelativeLayout
            android:id="@+id/relForgot"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_margin="20dp"
            >
            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Forgot Password"
                android:layout_alignParentEnd="true"
                android:layout_centerVertical="true"
                android:fontFamily="@font/robotobold"
                />
        </RelativeLayout>
        <Button
            android:id="@+id/btnLogin"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Login"
            android:fontFamily="@font/robotobold"
            android:layout_marginLeft="20dp"
            android:layout_marginRight="20dp"
            android:background="@color/primary"
            android:padding="15dp"
            android:layout_marginBottom="20dp"
            tools:ignore="VisualLintButtonSize" />
        <RelativeLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginLeft="20dp"
            android:layout_marginRight="20dp"
            >
            <TextView
                android:id="@+id/txtRegister"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Don't have an account? Register"
                android:layout_centerVertical="true"
                android:fontFamily="@font/robotobold"
                />
        </RelativeLayout>
    </LinearLayout>

</LinearLayout>
//Activity_register
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".auth.Register">

    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="56dp"
        >
        <LinearLayout
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            android:layout_centerVertical="true"
            android:layout_marginStart="18dp"
            tools:ignore="UselessParent">
            <ImageView
                android:id="@+id/registerBack"
                android:layout_width="30dp"
                android:layout_height="30dp"
                android:src="@drawable/backarrow"
                android:scaleType="fitXY"
                android:layout_gravity="center_vertical"
                app:tint="@android:color/holo_orange_light"
                android:contentDescription="TODO" />
            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@string/signup"
                android:fontFamily="@font/robotobold"
                android:textColor="@android:color/holo_orange_light"
                android:layout_gravity="center_vertical"
                android:textSize="20sp"
                android:layout_marginLeft="10dp"
                />

        </LinearLayout>

    </RelativeLayout>
    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        >
        <LinearLayout
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:layout_margin="20dp"
            >

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Sign up to continue chatting with friends within school"
                android:fontFamily="@font/robotoregular"
                android:layout_marginTop="10dp"
                android:textSize="16sp"
                android:textColor="@android:color/darker_gray"
                />

        </LinearLayout>
    </RelativeLayout>
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        >
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:padding="10dp">

            <EditText
                android:id="@+id/firstname"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:hint="@string/firstName"
                android:inputType="text"
                android:minHeight="48dp" />

            <EditText
                android:id="@+id/lastname"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:hint="@string/lastName"
                android:inputType="text"
                android:minHeight="48dp" />

            <EditText
                android:id="@+id/email"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:hint="@string/email"
                android:inputType="textEmailAddress"
                android:minHeight="48dp" />

            <EditText
                android:id="@+id/phone"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:hint="@string/phone"
                android:inputType="phone"
                android:maxLength="9"
                android:minHeight="48dp" />

            <EditText
                android:id="@+id/password"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:hint="Password"
                android:inputType="textPassword"
                android:minHeight="48dp" />

            <Button
                android:id="@+id/btnRegister"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="Register"
                android:padding="15dp"
                android:background="@color/primary"
                />

        </LinearLayout>

        <RelativeLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginLeft="20dp"
            android:layout_marginRight="20dp"
            >
            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Already have an account? Login"
                android:layout_centerVertical="true"
                android:fontFamily="@font/robotobold"
                />
        </RelativeLayout>
    </LinearLayout>

</LinearLayout>
//activity_splashScreen
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/teal_200"
    tools:context=".auth.SplashScreen">

    <LinearLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:layout_centerVertical="true"
        android:layout_centerHorizontal="true"
        >

        <RelativeLayout
            android:layout_width="130dp"
            android:layout_height="130dp"
            android:background="@drawable/back_image_splashscreen"
            android:layout_gravity="center"

            >
            <ImageView
                android:layout_width="90dp"
                android:layout_height="90dp"
                android:src="@drawable/chat_psut"
                android:scaleType="fitXY"
                android:layout_centerHorizontal="true"
                android:layout_centerVertical="true"
                />
        </RelativeLayout>
        <LinearLayout
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:layout_marginTop="300dp"
            >
            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="chatPSUT App"
                android:textColor="@color/white"
                android:textAlignment="center"
                android:layout_gravity="center"
                android:fontFamily="@font/robotobold"
                android:textSize="25sp"
                />
            <TextView
                android:layout_width="250dp"
                android:layout_height="wrap_content"
                android:text="@string/screen_tag"
                android:layout_gravity="center"
                android:textAlignment="center"
                android:layout_marginTop="10dp"
                android:textColor="@color/white"
                android:fontFamily="@font/robotoregular"

                />
        </LinearLayout>

    </LinearLayout>

</RelativeLayout>
//fragment_chat_room
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:background="@color/primary"
    tools:context=".chatclasses.ChatRoom">

    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="56dp"
        android:paddingLeft="10dp"
        android:paddingRight="10dp"
        android:background="@color/white"
        >
        <RelativeLayout
            android:id="@+id/relBackChats"
            android:layout_width="34dp"
            android:layout_height="34dp"
            android:layout_centerVertical="true"
            android:background="@drawable/back_chat_icons"
            >
            <ImageView
                android:layout_width="20dp"
                android:layout_height="20dp"
                android:src="@drawable/left"
                android:scaleType="fitXY"
                android:layout_centerVertical="true"
                android:layout_centerHorizontal="true"
                app:tint="@color/primary" />

        </RelativeLayout>

        <LinearLayout
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:layout_centerVertical="true"
            android:layout_toRightOf="@id/relBackChats"
            android:layout_marginLeft="15dp"

            >
            <TextView
                android:id="@+id/txtTitleChat"
                android:layout_width="280dp"
                android:layout_height="wrap_content"
                android:text="PSUT"
                android:fontFamily="@font/robotobold"
                android:textColor="@color/primary"
                android:textSize="16sp"
                android:maxLines="1"
                />
            <RelativeLayout
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                >
                <TextView
                    android:id="@+id/txtDesc"
                    android:layout_width="280dp"
                    android:layout_height="wrap_content"
                    android:text="Hakim"
                    android:ellipsize="end"
                    android:maxLines="1"
                    android:fontFamily="@font/robotoregular"
                    />
            </RelativeLayout>

        </LinearLayout>


        <RelativeLayout
            android:id="@+id/relBot"
            android:layout_width="34dp"
            android:layout_height="34dp"
            android:layout_alignParentEnd="true"
            android:layout_centerVertical="true"
            android:background="@drawable/back_chat_icons"
            >
            <ImageView
                android:layout_width="20dp"
                android:layout_height="20dp"
                android:src="@drawable/ic_baseline_more_vert_24"
                android:scaleType="fitXY"
                android:layout_centerHorizontal="true"
                android:layout_centerVertical="true"
                app:tint="@color/primary" />

        </RelativeLayout>

    </RelativeLayout>
    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        >
        <RelativeLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:background="@drawable/chat_rel_back"
            android:layout_above="@id/relBelow"

            >
            <androidx.recyclerview.widget.RecyclerView
                android:id="@+id/recyclerChats"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                tools:listitem="@layout/send_item"
                />

        </RelativeLayout>
        <RelativeLayout
            android:id="@+id/relBelow"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_alignParentBottom="true"
            android:background="@color/primary"
            >
            <LinearLayout
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:orientation="horizontal"
                android:background="@drawable/send_background"
                android:layout_margin="20dp"
                android:layout_centerHorizontal="true"
                android:layout_centerVertical="true"
                android:padding="12dp"
                >
                <ImageView
                    android:id="@+id/attach"
                    android:layout_width="30dp"
                    android:layout_height="30dp"
                    android:src="@drawable/ic_baseline_attach_file_24"
                    android:layout_marginRight="15dp"
                    android:layout_gravity="center_vertical"
                    />
                <LinearLayout
                    android:layout_width="260dp"
                    android:layout_height="wrap_content"
                    android:orientation="vertical"
                    android:layout_gravity="center_vertical"
                    >
                    <ImageView
                        android:id="@+id/imgChat"
                        android:layout_width="100dp"
                        android:layout_height="100dp"
                        android:src="@drawable/userchat"
                        android:scaleType="fitXY"
                        android:layout_marginBottom="5dp"
                        />
                    <EditText
                        android:id="@+id/editChat"
                        android:layout_width="260dp"
                        android:layout_height="wrap_content"
                        android:background="@drawable/edit_back"
                        android:hint="Enter Text"
                        android:textColor="@color/white"
                        android:textColorHint="@color/arrow_color"
                        android:fontFamily="@font/robotomedium"
                        />

                </LinearLayout>
                <ImageView
                    android:id="@+id/send"
                    android:layout_width="30dp"
                    android:layout_height="30dp"
                    android:src="@drawable/ic_baseline_send_24"
                    android:layout_marginLeft="15dp"
                    android:layout_gravity="center_vertical"
                    />

            </LinearLayout>

        </RelativeLayout>

    </RelativeLayout>



</LinearLayout>
//fragment_dashboard 
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".chatclasses.Dashboard">

    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="56dp"
        android:paddingLeft="10dp"
        android:paddingRight="10dp"
        >
        <RelativeLayout
            android:id="@+id/relProfile"
            android:layout_width="34dp"
            android:layout_height="34dp"
            android:layout_centerVertical="true"
            >
            <ImageView
                android:layout_width="20dp"
                android:layout_height="20dp"
                android:src="@drawable/userchat"
                android:scaleType="fitXY"
                android:layout_centerVertical="true"
                android:layout_centerHorizontal="true"
                />

        </RelativeLayout>
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="ChatRooms"
            android:layout_centerHorizontal="true"
            android:layout_centerVertical="true"
            android:fontFamily="@font/robotobold"
            android:textColor="@android:color/holo_orange_light"
            android:textSize="16sp"
            />

        <RelativeLayout
            android:id="@+id/relMore"
            android:layout_width="34dp"
            android:layout_height="34dp"
            android:layout_alignParentEnd="true"
            android:layout_centerVertical="true"
            android:background="@drawable/back_icons"
            >
            <ImageView
                android:layout_width="20dp"
                android:layout_height="20dp"
                android:src="@drawable/ic_baseline_more_vert_24"
                android:scaleType="fitXY"
                android:layout_centerHorizontal="true"
                android:layout_centerVertical="true"
                />

        </RelativeLayout>

    </RelativeLayout>
    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_marginTop="8dp"
        >

        <androidx.recyclerview.widget.RecyclerView
            android:id="@+id/recyclerGroups"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:layout_marginStart="10dp"
            android:layout_marginTop="10dp"
            android:layout_marginEnd="10dp"
            android:layout_marginBottom="10dp"
            tools:listitem="@layout/group_item" />

        <TextView
            android:id="@+id/txtHome"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Add ChatRoom"
            android:layout_centerHorizontal="true"
            android:layout_centerVertical="true"
            android:fontFamily="@font/robotobold"
            android:textSize="20sp"
            />


        <ImageButton
            android:id="@+id/floatDashboard"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_margin="20dp"
            android:background="@drawable/round_button"
            android:src="@drawable/ic_baseline_add_24"
            android:layout_alignParentBottom="true"
            android:layout_alignParentEnd="true"
            android:scaleType="center"/>

    </RelativeLayout>

</LinearLayout>
//fragment_new_chat_rooms
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@android:color/holo_orange_light"
    android:orientation="vertical"
    tools:context=".chatclasses.NewChatRooms">
    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="56dp"
        android:paddingLeft="10dp"
        android:paddingRight="10dp"
        >
        <RelativeLayout
            android:id="@+id/relBackNewChat"
            android:layout_width="34dp"
            android:layout_height="34dp"
            android:layout_centerVertical="true"
            android:background="@drawable/back_icons"
            >
            <ImageView
                android:layout_width="20dp"
                android:layout_height="20dp"
                android:src="@drawable/left"
                android:scaleType="fitXY"
                android:layout_centerVertical="true"
                android:layout_centerHorizontal="true"
                />

        </RelativeLayout>
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="New ChatRooms"
            android:layout_centerHorizontal="true"
            android:layout_centerVertical="true"
            android:fontFamily="@font/robotobold"
            android:textColor="@color/white"
            android:textSize="16sp"
            />

        <RelativeLayout
            android:id="@+id/relSearch"
            android:layout_width="34dp"
            android:layout_height="34dp"
            android:layout_alignParentEnd="true"
            android:layout_centerVertical="true"
            android:background="@drawable/back_icons"
            >
            <ImageView
                android:layout_width="20dp"
                android:layout_height="20dp"
                android:src="@drawable/searchchat"
                android:scaleType="fitXY"
                android:layout_centerHorizontal="true"
                android:layout_centerVertical="true"
                />

        </RelativeLayout>

    </RelativeLayout>
    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@drawable/recycler_back"
        android:layout_marginTop="8dp"
        >
        <androidx.recyclerview.widget.RecyclerView
            android:id="@+id/recyclerNewChatRoom"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:layout_margin="10dp"
            tools:listitem="@layout/new_chat_item"
            />

        <TextView
            android:id="@+id/txtAvailable"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="No Available ChatRoom"
            android:layout_centerHorizontal="true"
            android:layout_centerVertical="true"
            android:fontFamily="@font/robotobold"
            android:textSize="20sp"
            />

    </RelativeLayout>

</LinearLayout>
//fragment_profile
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:background="@android:color/holo_orange_light"
    tools:context=".chatclasses.Profile">

    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="56dp"
        android:paddingLeft="10dp"
        android:paddingRight="10dp"
        >
        <RelativeLayout
            android:id="@+id/relBackProf"
            android:layout_width="34dp"
            android:layout_height="34dp"
            android:layout_centerVertical="true"
            android:background="@drawable/back_icons"
            >
            <ImageView
                android:layout_width="20dp"
                android:layout_height="20dp"
                android:src="@drawable/left"
                android:scaleType="fitXY"
                android:layout_centerVertical="true"
                android:layout_centerHorizontal="true"
                />

        </RelativeLayout>
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Profile"
            android:layout_centerHorizontal="true"
            android:layout_centerVertical="true"
            android:fontFamily="@font/robotobold"
            android:textColor="@color/white"
            android:textSize="16sp"
            />

        <RelativeLayout
            android:id="@+id/logOut"
            android:layout_width="34dp"
            android:layout_height="34dp"
            android:layout_alignParentEnd="true"
            android:layout_centerVertical="true"
            android:background="@drawable/back_icons"
            >
            <ImageView
                android:layout_width="20dp"
                android:layout_height="20dp"
                android:src="@drawable/ic_baseline_login_24"
                android:scaleType="fitXY"
                android:layout_centerHorizontal="true"
                android:layout_centerVertical="true"
                />

        </RelativeLayout>

    </RelativeLayout>

    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@drawable/recycler_back"
        >
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:orientation="vertical"
            >
            <RelativeLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginLeft="25dp"
                android:layout_marginRight="25dp"
                android:layout_marginBottom="10dp"
                android:layout_marginTop="35dp"
                >
                <TextView
                    android:id="@+id/firstN"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="First Name"
                    android:fontFamily="@font/robotoregular"
                    android:textColor="@color/black"
                    android:textSize="17sp"
                    android:layout_centerVertical="true"
                    />
                <ImageView
                    android:layout_width="15dp"
                    android:layout_height="15dp"
                    android:src="@drawable/ic_baseline_arrow_forward_ios_24"
                    app:tint="@color/primary"
                    android:layout_centerVertical="true"
                    android:layout_alignParentEnd="true"/>
            </RelativeLayout>
            <RelativeLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginLeft="25dp"
                android:layout_marginRight="25dp"
                android:layout_marginBottom="10dp"
                android:layout_marginTop="20dp"
                >
                <TextView
                    android:id="@+id/lastN"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="Last Name"
                    android:fontFamily="@font/robotoregular"
                    android:textColor="@color/black"
                    android:textSize="17sp"
                    android:layout_centerVertical="true"
                    />
                <ImageView
                    android:layout_width="15dp"
                    android:layout_height="15dp"
                    android:src="@drawable/ic_baseline_arrow_forward_ios_24"
                    app:tint="@color/primary"
                    android:layout_centerVertical="true"
                    android:layout_alignParentEnd="true"/>
            </RelativeLayout>

            <RelativeLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginLeft="25dp"
                android:layout_marginRight="25dp"
                android:layout_marginBottom="10dp"
                android:layout_marginTop="20dp"
                >
                <TextView
                    android:id="@+id/em"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="Email"
                    android:fontFamily="@font/robotoregular"
                    android:textColor="@color/black"
                    android:textSize="17sp"
                    android:layout_centerVertical="true"
                    />
                <ImageView
                    android:layout_width="15dp"
                    android:layout_height="15dp"
                    android:src="@drawable/ic_baseline_arrow_forward_ios_24"
                    app:tint="@color/primary"
                    android:layout_centerVertical="true"
                    android:layout_alignParentEnd="true"/>
            </RelativeLayout>

            <RelativeLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginLeft="25dp"
                android:layout_marginRight="25dp"
                android:layout_marginBottom="10dp"
                android:layout_marginTop="20dp"
                >
                <TextView
                    android:id="@+id/phoneProf"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="Phone"
                    android:fontFamily="@font/robotoregular"
                    android:textColor="@color/black"
                    android:textSize="17sp"
                    android:layout_centerVertical="true"
                    />
                <ImageView
                    android:layout_width="15dp"
                    android:layout_height="15dp"
                    android:src="@drawable/ic_baseline_arrow_forward_ios_24"
                    app:tint="@color/primary"
                    android:layout_centerVertical="true"
                    android:layout_alignParentEnd="true"/>
            </RelativeLayout>

        </LinearLayout>

    </RelativeLayout>



</LinearLayout>
//fragment_search 
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:background="@android:color/holo_orange_light"
    tools:context=".chatclasses.Search">

    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="56dp"
        android:paddingLeft="10dp"
        android:paddingRight="10dp"
        >
        <RelativeLayout
            android:id="@+id/relBackSearch"
            android:layout_width="34dp"
            android:layout_height="34dp"
            android:layout_centerVertical="true"
            android:background="@drawable/back_icons"
            >
            <ImageView
                android:layout_width="20dp"
                android:layout_height="20dp"
                android:src="@drawable/left"
                android:scaleType="fitXY"
                android:layout_centerVertical="true"
                android:layout_centerHorizontal="true"
                />

        </RelativeLayout>
        <EditText
            android:id="@+id/editSearch"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_toRightOf="@id/relBackSearch"
            android:layout_marginLeft="10dp"
            android:layout_marginRight="10dp"
            android:hint="Search Chatroom"
            android:background="@drawable/search_back"
            android:layout_centerVertical="true"
            android:padding="6dp"
            android:textColor="@color/white"
            android:textColorHint="@color/white"
            />

    </RelativeLayout>

    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@drawable/recycler_back"
        >
        <androidx.recyclerview.widget.RecyclerView
            android:id="@+id/recSearch"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:padding="20dp"
            />

    </RelativeLayout>


</LinearLayout>
//fragment_verification
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".chatclasses.Verification">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:layout_centerVertical="true"
        android:layout_centerHorizontal="true"
        >
        <RelativeLayout
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center_horizontal"
            >

        </RelativeLayout>

        <RelativeLayout
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center_horizontal"
            android:layout_marginTop="40dp"
            >
            <LinearLayout
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:orientation="vertical"
                >
                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="Code Verification"
                    android:textColor="@android:color/holo_orange_light"
                    android:fontFamily="@font/robotobold"
                    android:textSize="20sp"
                    android:textAlignment="center"
                    android:layout_gravity="center_horizontal"
                    />
                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="Enter Code sent to"
                    android:fontFamily="@font/robotoregular"
                    android:layout_gravity="center_horizontal"
                    android:layout_marginTop="8dp"
                    android:layout_marginBottom="8dp"
                    />
                <TextView
                    android:id="@+id/phoneNumber"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:textColor="@color/black"
                    android:fontFamily="@font/robotomedium"
                    android:layout_gravity="center_horizontal"
                    />

            </LinearLayout>
        </RelativeLayout>
        <RelativeLayout
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center_horizontal"
            android:layout_marginTop="15dp"
            >
            <LinearLayout
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:orientation="horizontal"
                >
                <EditText
                    android:id="@+id/editOne"
                    android:layout_width="30dp"
                    android:layout_height="wrap_content"
                    android:textAlignment="center"
                    android:textStyle="bold"
                    android:fontFamily="@font/robotobold"
                    android:maxLength="1"
                    android:backgroundTint="@color/black"
                    />
                <EditText
                    android:id="@+id/editTwo"
                    android:layout_width="30dp"
                    android:layout_height="wrap_content"
                    android:textAlignment="center"
                    android:textStyle="bold"
                    android:maxLength="1"
                    android:fontFamily="@font/robotobold"
                    android:backgroundTint="@color/black"
                    />
                <EditText
                    android:id="@+id/editThree"
                    android:layout_width="30dp"
                    android:layout_height="wrap_content"
                    android:textAlignment="center"
                    android:textStyle="bold"
                    android:fontFamily="@font/robotobold"
                    android:backgroundTint="@color/black"
                    />
                <EditText
                    android:id="@+id/editFour"
                    android:layout_width="30dp"
                    android:layout_height="wrap_content"
                    android:maxLength="1"
                    android:textAlignment="center"
                    android:textStyle="bold"
                    android:fontFamily="@font/robotobold"
                    android:backgroundTint="@color/black"
                    />
                <EditText
                    android:id="@+id/editFive"
                    android:layout_width="30dp"
                    android:layout_height="wrap_content"
                    android:maxLength="1"
                    android:textAlignment="center"
                    android:textStyle="bold"
                    android:fontFamily="@font/robotobold"
                    android:backgroundTint="@color/black"
                    />
                <EditText
                    android:id="@+id/editSix"
                    android:layout_width="30dp"
                    android:layout_height="wrap_content"
                    android:maxLength="1"
                    android:textAlignment="center"
                    android:textStyle="bold"
                    android:fontFamily="@font/robotobold"
                    android:backgroundTint="@color/black"
                    />

            </LinearLayout>
        </RelativeLayout>
        <LinearLayout
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            android:layout_gravity="center_horizontal"
            android:layout_marginTop="15dp"
            >
            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Didn't receive the code? "
                android:fontFamily="@font/robotomedium"
                />
            <TextView
                android:id="@+id/resend"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Resend Code"
                android:fontFamily="@font/robotobold"
                android:textColor="@color/design_default_color_error"
                />

        </LinearLayout>

        <Button
            android:id="@+id/btnCode"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Verify"
            android:layout_marginStart="30dp"
            android:layout_marginEnd="30dp"
            android:layout_marginTop="15dp"
            android:padding="13dp"
            android:fontFamily="@font/robotobold"
            android:background="@android:color/holo_orange_light"
            android:textColor="@color/white"
            />

    </LinearLayout>

</RelativeLayout>
//group_item
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:orientation="horizontal"
    android:id="@+id/groupId"
    android:padding="15dp"
    android:layout_height="wrap_content">

    <LinearLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        >
        <androidx.cardview.widget.CardView
            android:layout_width="50dp"
            android:layout_height="50dp"
            app:cardCornerRadius="50dp"
            android:layout_gravity="center_vertical"
            android:padding="2dp"
            >
            <RelativeLayout
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:layout_gravity="center_vertical"
                >
                <ImageView
                    android:id="@+id/imgGroupImage"
                    android:layout_width="50dp"
                    android:layout_height="50dp"
                    android:scaleType="fitXY"
                    android:src="@drawable/images"
                    android:layout_centerVertical="true"
                    android:layout_centerHorizontal="true"
                    />
            </RelativeLayout>
        </androidx.cardview.widget.CardView>

        <LinearLayout
            android:layout_width="260dp"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:layout_gravity="center_vertical"
            android:layout_marginLeft="13dp"
            >
            <TextView
                android:id="@+id/groupName"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Princess Summaya University of technology"
                android:textColor="@color/black"
                android:fontFamily="@font/robotomedium"
                android:maxLines="1"
                android:ellipsize="end"
                />
            <TextView
                android:id="@+id/groupMessage"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Hey we have a class in the next few minutes"
                android:fontFamily="@font/robotoregular"
                android:layout_marginTop="2dp"
                android:maxLines="1"
                android:ellipsize="end"
                />
        </LinearLayout>

    </LinearLayout>
    <LinearLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:layout_alignParentEnd="true"
        >
        <TextView
            android:id="@+id/groupTime"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="7:30 PM"
            android:layout_gravity="center_horizontal"
            android:textSize="10sp"
            android:fontFamily="@font/robotomedium"
            />

        <RelativeLayout
            android:layout_width="16dp"
            android:layout_height="16dp"
            android:background="@drawable/total_back"
            android:layout_gravity="center_horizontal"
            android:layout_marginTop="10dp"
            >
            <TextView
                android:id="@+id/groupTotal"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="5"
                android:layout_centerVertical="true"
                android:layout_centerHorizontal="true"
                android:textColor="@color/white"
                android:fontFamily="@font/robotomedium"
                android:textSize="8sp"
                />

        </RelativeLayout>
    </LinearLayout>

</RelativeLayout>
//leave_sheet
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:background="@android:color/holo_orange_light"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:layout_margin="15dp"
        >
        <View
            android:layout_width="40dp"
            android:layout_height="5dp"
            android:layout_gravity="center"
            android:background="@drawable/round_button"
            android:layout_marginTop="10dp"
            />

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:layout_margin="20dp"
            >
            <LinearLayout
                android:id="@+id/linLeave"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:orientation="horizontal"
                android:layout_gravity="center_vertical"
                >
                <ImageView
                    android:layout_width="20dp"
                    android:layout_height="20dp"
                    android:src="@drawable/ic_baseline_login_24"
                    android:layout_gravity="center_vertical"
                    android:scaleType="fitXY"
                    app:tint="@color/purple_700" />
                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="Leave Chatroom"
                    android:fontFamily="@font/robotomedium"
                    android:layout_marginLeft="10dp"
                    android:layout_gravity="center_vertical"
                    />

            </LinearLayout>

        </LinearLayout>

    </LinearLayout>

</RelativeLayout>
//new_chat_item 
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:background="@color/white"
    android:padding="15dp"
    android:layout_height="wrap_content">

    <LinearLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        >
        <androidx.cardview.widget.CardView
            android:layout_width="50dp"
            android:layout_height="50dp"
            app:cardCornerRadius="50dp"
            android:layout_gravity="center_vertical"
            android:padding="2dp"
            >
            <RelativeLayout
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:layout_gravity="center_vertical"
                >
                <ImageView
                    android:id="@+id/imgChatImage"
                    android:layout_width="40dp"
                    android:layout_height="40dp"
                    android:scaleType="fitXY"
                    android:src="@drawable/logo"
                    android:layout_centerVertical="true"
                    android:layout_centerHorizontal="true"
                    />
            </RelativeLayout>
        </androidx.cardview.widget.CardView>
        <LinearLayout
            android:layout_width="230dp"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:layout_gravity="center_vertical"
            android:layout_marginLeft="13dp"
            >
            <TextView
                android:id="@+id/groupChatName"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Mobile Applications"
                android:textColor="@color/black"
                android:fontFamily="@font/robotomedium"
                android:maxLines="1"
                android:ellipsize="end"
                />
            <TextView
                android:id="@+id/groupChatDescription"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Join to room"
                android:fontFamily="@font/robotoregular"
                android:layout_marginTop="2dp"
                android:maxLines="1"
                android:ellipsize="end"
                />
        </LinearLayout>

    </LinearLayout>
    <LinearLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:layout_alignParentEnd="true"
        >


        <RelativeLayout
            android:id="@+id/relJoin"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:background="@drawable/chat_back"
            android:layout_gravity="center_horizontal"
            android:layout_marginTop="10dp"
            android:padding="3dp"
            >
            <TextView
                android:id="@+id/groupTotal"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Join Chatroom"
                android:layout_centerVertical="true"
                android:layout_centerHorizontal="true"
                android:textColor="@color/white"
                android:fontFamily="@font/robotomedium"
                android:textSize="10sp"
                />

        </RelativeLayout>
    </LinearLayout>

</RelativeLayout>
//newchatroom_bottom_sheet
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:background="@color/white"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:layout_margin="10dp"
        >
        <View
            android:layout_width="40dp"
            android:layout_height="5dp"
            android:layout_gravity="center"
            android:background="@drawable/round_button"
            android:layout_marginTop="10dp"
            />

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:layout_margin="20dp"
            >
            <RelativeLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                >
                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="Add ChatRoom"
                    android:fontFamily="@font/robotobold"
                    android:layout_centerHorizontal="true"
                    android:layout_centerVertical="true"
                    android:textColor="@color/primary"
                    />
            </RelativeLayout>
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical"
                >
                <RelativeLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:layout_margin="15dp"
                    >
                    <androidx.cardview.widget.CardView
                        android:layout_width="50dp"
                        android:layout_height="50dp"
                        app:cardCornerRadius="50dp"
                        android:layout_centerHorizontal="true"
                        android:layout_gravity="center_vertical"
                        android:padding="2dp"
                        >
                        <RelativeLayout
                            android:layout_width="match_parent"
                            android:layout_height="match_parent"
                            android:layout_gravity="center_vertical"
                            >
                            <ImageView
                                android:id="@+id/imgChatRoomPic"
                                android:layout_width="40dp"
                                android:layout_height="40dp"
                                android:scaleType="fitXY"
                                android:src="@drawable/logo"
                                android:layout_centerVertical="true"
                                android:layout_centerHorizontal="true"
                                />
                        </RelativeLayout>
                    </androidx.cardview.widget.CardView>

                </RelativeLayout>
                <RelativeLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:layout_marginBottom="5dp"
                    >
                    <TextView
                        android:id="@+id/chatPic"
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:text="Add Chatroom photo"
                        android:layout_centerHorizontal="true"
                        android:fontFamily="@font/robotomedium"
                        />
                </RelativeLayout>
                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="vertical"
                    android:padding="10dp">

                    <EditText
                        android:id="@+id/chatName"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="10dp"
                        android:hint="Chatroom Title"
                        android:inputType="text" />

                    <EditText
                        android:id="@+id/chatDescription"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="10dp"
                        android:hint="Chatroom Description"
                        android:inputType="text" />

                    <EditText
                        android:id="@+id/chatCapacity"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="10dp"
                        android:hint="Chatroom Maximum Capacity"
                        android:inputType="number" />

                    <Button
                        android:id="@+id/btnChat"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="13dp"
                        android:layout_marginBottom="10dp"
                        android:backgroundTint="@color/primary"
                        android:text="Add"
                        android:padding="15dp"
                        />

                </LinearLayout>

            </LinearLayout>
            <RelativeLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                >
                <ProgressBar
                    android:id="@+id/progressNew"
                    android:layout_width="20dp"
                    android:layout_height="20dp"
                    android:layout_centerHorizontal="true"
                    />
            </RelativeLayout>
        </LinearLayout>

    </LinearLayout>

</RelativeLayout>
//options_buttom_sheet 
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:background="@drawable/back_icons"
    android:layout_height="wrap_content">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:layout_margin="15dp"
        >
        <View
            android:layout_width="40dp"
            android:layout_height="5dp"
            android:layout_gravity="center"
            android:background="@drawable/send_item_background"
            android:layout_marginTop="10dp"
            />

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:layout_margin="20dp"
            >
            <LinearLayout
                android:id="@+id/linJoin"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:orientation="horizontal"
                android:layout_gravity="center_vertical"
                >
                <ImageView
                    android:layout_width="20dp"
                    android:layout_height="20dp"
                    android:src="@drawable/newjoin"
                    android:layout_gravity="center_vertical"
                    android:scaleType="fitXY"
                    />
                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="Join new chatroom"
                    android:fontFamily="@font/robotomedium"
                    android:layout_marginLeft="10dp"
                    android:layout_gravity="center_vertical"
                    />

            </LinearLayout>
            <LinearLayout
                android:id="@+id/linSearch"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:orientation="horizontal"
                android:layout_gravity="center_vertical"
                android:layout_marginTop="20dp"
                >
                <ImageView
                    android:layout_width="20dp"
                    android:layout_height="20dp"
                    android:src="@drawable/searchchat"
                    android:layout_gravity="center_vertical"
                    android:scaleType="fitXY"
                    app:tint="@color/primary" />
                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="Search chatroom"
                    android:fontFamily="@font/robotomedium"
                    android:layout_marginLeft="10dp"
                    android:layout_gravity="center_vertical"
                    />

            </LinearLayout>

        </LinearLayout>

    </LinearLayout>

</RelativeLayout>
//receive_item 
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">
    <LinearLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:paddingLeft="10dp"
        android:paddingRight="10dp"
        >
        <LinearLayout
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:background="@drawable/back_image_splashscreen"
            android:padding="10dp"
            >
            <TextView
                android:id="@+id/recName"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Hakim"
                android:fontFamily="@font/robotoregular"
                android:textColor="@android:color/darker_gray"
                android:textSize="12sp"
                />
            <ImageView
                android:id="@+id/imgReceiveMessage"
                android:layout_width="100dp"
                android:layout_height="100dp"
                android:src="@drawable/userchat"
                android:scaleType="fitXY"
                android:layout_marginTop="5dp"
                />
            <TextView
                android:id="@+id/recContent"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Hello, I have been looking for you"
                android:textColor="@color/primary"
                android:fontFamily="@font/robotomedium"
                android:layout_marginTop="5dp"
                />
        </LinearLayout>
        <RelativeLayout
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="start"
            android:padding="5dp"
            >
            <TextView
                android:id="@+id/recTime"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="7:15 PM"
                android:fontFamily="@font/robotomedium"
                android:textSize="10sp"
                />
        </RelativeLayout>

    </LinearLayout>

</RelativeLayout>
//send_item 
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

    <LinearLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:layout_alignParentEnd="true"
        android:paddingLeft="10dp"
        android:paddingRight="10dp"
        >
        <LinearLayout
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:background="@drawable/back_icons"
            android:padding="10dp"
            >
            <TextView
                android:id="@+id/nmChat"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Hakim"
                android:fontFamily="@font/robotoregular"
                android:textColor="@color/lightblue"
                android:textSize="12sp"
                />
            <ImageView
                android:id="@+id/imgChatMessage"
                android:layout_width="100dp"
                android:layout_height="100dp"
                android:src="@drawable/userchat"
                android:scaleType="fitXY"
                android:layout_marginTop="5dp"
                />
            <TextView
                android:id="@+id/chatContent"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Hello, I have been looking for you"
                android:textColor="@color/white"
                android:fontFamily="@font/robotomedium"
                android:layout_marginTop="5dp"
                />
        </LinearLayout>
        <RelativeLayout
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="end"
            android:padding="5dp"
            >
            <TextView
                android:id="@+id/chatTime"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="7:15 PM"
                android:fontFamily="@font/robotomedium"
                android:textSize="10sp"
                />
        </RelativeLayout>

    </LinearLayout>

</RelativeLayout>

